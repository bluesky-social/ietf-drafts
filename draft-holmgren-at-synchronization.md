---
title: "Authenticated Transfer Synchronization"
abbrev: "AT Sync"
category: std

docname: draft-holmgren-at-synchronization-latest
submissiontype: IETF
number:
updates:
date:
consensus:
v: 0
area: "Applications and Real-Time Area"
workgroup:
keyword:
venue:
  github: "bluesky-social/ietf-drafts"

author:
 -
    fullname: Daniel Holmgren
    organization: Bluesky Social
    email: daniel@blueskyweb.xyz
 -
    fullname: Bryan Newbold
    organization: Bluesky Social
    email: bryan@blueskyweb.xyz

normative:
  RFC6455:
  ATREPO:
    title: "Authenticated Transfer Repository"
    date: draft-holmgren-at-repository
    author:
      -
        fullname: Daniel Holmgren
        organization: Bluesky Social
      -
        fullname: Bryan Newbold
        organization: Bluesky Social

informative:
  MST:
    title: "Merkle Search Trees: Efficient State-Based CRDTs in Open Networks"
    date: October 2019
    target: https://inria.hal.science/hal-02303490/document
    author:
      -
        fullname: Alex Auvolat
      -
        fullname: François Taïani
  AT-ARCH:
    title: "Authenticated Transfer: Architecture Overview"
    date: draft-newbold-at-architecture
    author:
      -
        fullname: Bryan Newbold
        organization: Bluesky Social
      -
        fullname: Daniel Holmgren
        organization: Bluesky Social
...

--- abstract

This document specifies the synchronization protocol for Authenticated Transfer (AT), a protocol for cryptographically-verifiable storage and distribution of structured user-controlled data. It defines synchronization mechanisms that allow efficient distribution of repository changes to interested parties.

This document builds on the AT repository format specified in {{ATREPO}}. Overall network architecture is described further in {{AT-ARCH}}.

--- middle

# Introduction {#intro}

The Authenticated Transfer (AT) synchronization protocol addresses the challenges of building decentralized applications that require consistent data replication across distributed multi-party infrastructure. In the AT model, user data is stored in cryptographically signed repositories as specified in {{ATREPO}}. The synchronization system provides efficient mechanisms for propagating repository state changes across the network, supporting both real-time streaming updates and bulk synchronization scenarios. The protocol can detect dropped or withheld updates and provides cryptographic proofs for all operations, including record deletions, ensuring that consumers can maintain accurate and complete views of repository state.

The AT synchronization model operates on the principle that any participant can independently verify repository updates. This allows sync to occur between any client and server without requiring that the server is a canonical or trusted host.

AT supports multiple synchronization patterns: full repository synchronization for complete replicas, partial synchronization for specific record subsets, and proof-only synchronization for cryptographic verification without content retrieval.

The typical synchronization workflow establishes baseline state through full synchronization, then maintains currency through incremental updates. Full synchronization is performed by fetching a complete serialized repository over HTTPS, as specified in {{ATREPO}}.

Repositories carry a revision identifier (`rev`) on each commit which functions as a logical clock and must increase monotonically. Revisions may be used when comparing two repositories, especially when obtained from a non-canonical host, to determine which is more recent. The repository revision format, including the recommended Timestamp Identifier (TID) encoding, is specified in {{ATREPO}}.

# Repository Diffs {#diffs}

Repository diffs enable efficient synchronization by containing only the data that changed between two repository revisions. A diff includes the commit object, MST nodes, and records that differ between an older baseline revision and the current revision. Applying a diff to the baseline repository reconstructs the complete current repository state.

Diffs use the same serialization format as complete repositories, with the commit block serving as the root. A diff must include:

- The new commit block
- All created and updated record blocks
- All MST nodes in the current repository that did not exist in the baseline revision

Required blocks must be included in the diff regardless of their presence in earlier repository history. For example, if an MST node was previously present in the repository, then deleted, and subsequently reintroduced during the range that the diff represents, then the diff must include that block even though it appeared in prior revisions.

Deleted records and past versions of updated records are excluded from diffs.

With the exception of deleted record data, the diff may include additional blocks which receivers should ignore.

# Diff Verification Limitations {#diff-limits}

Repository diffs present verification challenges for consumers who do not maintain complete repository state. These consumers often wish to authenticate repository content and utilize records without persisting the entire repository structure, making diffs an attractive option for lightweight verification.

Diffs partially support this use case by providing a signed commit and the relevant portions of the Merkle tree, creating a verifiable proof chain for record creations, updates, and deletions. When a recipient possesses both a diff and a corresponding list of operations, they can use the diff contents to cryptographically verify that the operations are authentic.

However, observers without knowledge of the complete baseline repository state cannot reliably enumerate all operations by examining the diff contents alone. While comprehensive diffs  reveal created or updated records by traversing to the leaf nodes, they provide no information about deletion operations that occurred during the period that the diff represents.

This means that while diffs enable verification of a known operation list, they cannot be used to exhaustively reconstruct the complete operation list from diff contents alone. However, if a recipient has a complete repository structure from some prior revision and receives a diff representing changes since that revision, they can compute the complete set of operations that occurred between the two versions.

This asymmetry means diffs alone cannot substitute for complete state tracking when comprehensive operation enumeration is required. An efficient mechanism for cross-verification of a diff and enumerated operation list against the prior repository commit state is described in {{streaming-validation}}.

# Real-time synchronization {#realtime}

AT supports real-time synchronization, enabling applications to receive repository updates with minimal latency through a pull-based WebSocket {{RFC6455}} connection.

Real-time streams of repository updates are often referred to as the “firehose”. The firehose delivers events containing repository diffs along with supporting metadata necessary for verification and processing.

Each event includes a monotonic cursor that establishes a total ordering across all repository changes from a given host. This ordering enables reliable event replay and ensures that consumers can maintain consistent state even when reconnecting after network interruptions.

AT allows consumers to maintain fully-verified copies of repository records without storing the underlying Merkle tree structure, providing an efficient method for applications that need authenticated content access without the overhead of complete repository replication.

## Cursors {#cursors}

Real-time synchronization streams include per-message cursors to improve transmission reliability. Cursors are positive integers that increase monotonically across the stream. Cursor semantics are flexible, and they may contain arbitrary gaps between consecutive messages.

Consumers track the last cursor value they successfully processed and can specify this cursor when reconnecting to receive any missed messages within the provider's rollback window. Providers maintain no persistent consumer state across connections, relying entirely on the cursor values supplied by consumers during reconnection.

Stream behavior depends on the cursor value specified during connection:

- **No cursor specified**: The provider begins transmitting from the current stream position, providing only new messages generated after the connection is established.
- **Future cursor**: When the requested cursor exceeds the current stream cursor, the provider sends an error message and closes the connection.
- **Cursor within rollback window**: The provider transmits all persisted messages with cursor numbers greater than or equal to the requested cursor, then continues with the real-time stream once caught up.
- **Cursor older than rollback window**: The provider sends an informational message indicating that the requested cursor is too old, then begins transmission at the oldest available event, sends the entire rollback window, and continues with the real-time stream.
- **Cursor value of 0**: The provider treats this as a request for the complete available history, starting at the oldest available event, transmitting the entire rollback window, then continuing with the real-time stream.

## Streaming Events {#streaming-events}

The real-time stream delivers two types of events: `commit` and `sync`.

### Commit Events {#commit-events}

Commit events represent an atomic set of repository modifications and consist of a repository diff combined with some supporting metadata.

The diff MUST include the new commit and all blocks in the Merkle proof chain for any modified key, as well as blocks for keys directly adjacent to the modified keys. The rationale for including adjacent keys is detailed in {{streaming-validation}}.

The metadata provides additional context required for processing and verification and includes:

- The revision of the repository after the modifications
- The revision of the repository before the diff
- The root hash of the repository MST before the diff
- A description of the operations contained in the diff with each containing
    - the key
    - the hash of the new record at the key (in the case of a create/update)
    - the hash of the old record at the key (in the case of an update/delete)

A single commit events must contain no more than 200 repository operations and the full serialized event should be no larger than 2MB. Mutations that do not fit in these limits should instead be communicated through Sync Events.

### Sync Events {#sync-events}

Sync events declare the current state of a repository, regardless of the previous state.

Sync events are emitted when commit events cannot adequately describe the transition between repository revisions. This may occur in several scenarios:

- Large mutations that exceed the practical size limits for commit events
- Data loss or corruption that breaks the continuity of commit history
- Account migration between different infrastructure providers

In these cases, a sync event provides a reset point that encourages consumers to resynchronize against the current authoritative state without requiring knowledge of the intervening changes.

## Commit Validation {#streaming-validation}

Commit validation occurs through a two-step process that ensures both the validity of the repository transition and the consumer's resulting synchronization state.

First, the consumer validates that the commit represents a valid transition from a previous repository revision (`revA`) to the new revision (`revB`). Second, the consumer confirms that they last observed the repository at `revA`. Together, these steps establish that the repository is now definitively at `revB`.

The validation process inverts all operations against the partial MST provided in the diff. That is, each “create” operation will be inverted as a “delete” operation on the same key and applied to the tree. Each “delete” will become a “create” of the same record, and every “update” will be updated back to the previous value.

If the operation list is complete and accurate, applying the inverse operations will reconstruct the tree state as it existed before the commit. The hash of this reconstructed tree must match the previous root hash of the MST as specified in the commit event. If the hashes match then the provided list of operations is accurate and exhaustive.

Because the previous MST root hash is included in the commit event, commits can be validated for internal consistency independent of any local state. If the operation inversion process fails to produce a tree hash matching the declared previous root, the entire commit event should be treated as invalid.

If the commit is internally consistent but its declared previous root does not match the previous MST root stored locally, then the consumer has become desynchronized, indicating missed events or a disjunction in the producer’s commit history.

## Re-synchronization {#resync}

When a consumer detects desynchronization, either through a disjunction in commit history or a `sync` event that does not match their local state, they must perform a complete re-synchronization process to restore consistency with the current repository state.

Re-synchronization requires fetching and processing the full repository structure, though the record contents themselves are optional depending on the consumer's needs. If the repository data is delivered in pre-order traversal, it can be validated incrementally as it streams in, producing a mapping of keys to record hashes that represents the complete repository state.

This key-to-hash mapping can be compared against existing local state to identify discrepancies and verify the integrity of the re-synchronization. Once validated, this mapping establishes the new baseline state against which future commit events can be applied.

During the re-synchronization process, any incoming commit events for the repository should be buffered rather than processed immediately. Once re-synchronization completes successfully, these buffered commits can be validated and applied in sequence to bring the consumer fully up to date with the current repository state.

# Security Considerations {#security}

General security considerations for the repository format itself, including CBOR processing limits and MST structural validation, are covered in {{ATREPO}}.

## Resource Abuse {#security-resources}

A producer that routinely emits sync events forces consumers into full re-synchronization, which is substantially more expensive than processing commit events. Similarly, a producer that rapidly updates the same key or issues many small commits can amplify event volume and bandwidth costs on downstream consumers. Consumers should apply rate limits and bandwidth budgets per repository, and may disconnect or deprioritize producers whose event patterns appear abusive.

## Repository Rewinds {#security-rewinds}

Intermediaries that relay firehose events can present a consumer with an outdated view of a repository by replaying older commits or declining to forward newer ones. The `rev` field on each commit can help consumers detect this: a received commit whose `rev` is not greater than the most recently observed `rev` for that repository may indicate that the producer is serving a rewound view. In situations such as this, consumers can cross-check the latest observed `rev` against the canonical host for the repository.
