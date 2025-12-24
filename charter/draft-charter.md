Authenticated Transfer Protocol (ATP) Working Group
===================================================

The Authenticated Transfer Protocol (ATP) working group will develop a set of standards-track specifications that enable creation of an interoperable global network for publication of self-certifying data. The protocol enables synchronization of public data across organizational boundaries, while preserving authenticity of content and attribution. The network will give small independent participants global visibility and distribution without prior coordination, and will not depend on the existence of any single privileged entity.

The work will be based on draft-holmgren-at-repository and the pre-standardization implementation and deployment experience of multiple parties in the ATP ecosystem. All existing work must undergo a consensus process, and changes may be necessary, but gratuitous changes will be avoided when possible.

The working group will focus on a general purpose data repository structure for public user data records, and mechanisms for synchronizing repositories across the network. Repositories are cryptographically verifiable and publicly published.

Cryptographic verification of repository signatures requires a system for resolving account identifiers to public signing keys. Account identifiers are also used to resolve the current hosting location. Account identifiers should be globally resolvable and persist across hosting migrations. The working group will specify interface requirements and selection criteria for account identifier systems. To ensure broad interoperation in the global network, the working group will maintain a list indicating which identifier systems are recommended as a baseline. Backwards compatibility will be maintained for the two account identifier systems supported in the currently deployed network (PLC and did:web).

Key goals for the working group:

- A low-latency streaming synchronization protocol, including notification of data deletion
- The streaming protocol should allow for alternative network transports
- Scale to global network applications
- Support for redistribution of authenticated data via multiple layers of intermediary parties
- Encoding format for structured data records (CBOR and JSON representations)
- Export format for transfer of data repositories
- URI scheme for persistent references between data records
- Account migration between network locations (eg, hosting providers) does not break references or graph relationships

The following are out of scope for the working group:

- Transfer of non-public or limited-visibility data
- Any application-specific data schemas
- Any social application semantics

The ATP working group will consult the Crypto Forum Research Group (CFRG) and Security Area groups concerning the use and selection of any cryptographic systems. Privacy and Centralization considerations will be documented (eg, per RFC 6873) and will inform design decisions.

