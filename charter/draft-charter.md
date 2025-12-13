Authenticated Transfer Protocol (ATP) Working Group
===================================================

The Authenticated Transfer Protocol (ATP) working group will develop a set of standards-track specifications that enable creation of an interoperable global network for publication of self-certifying data. The protocol enables synchronization of public data across organizational boundaries, while preserving authenticity of content and attribution. The network will give small independent participants global visibility and distribution without prior coordination, and will not depend on the existence of any single privileged entity.

The work will be based on draft-holmgren-at-repository and the pre-standardization implementation and deployment experience of multiple parties in the ATP ecosystem. All existing work must undergo a consensus process, and changes may be necessary, but gratuitous changes will be avoided when possible.

The working group will focus on a general purpose data repository structure for public user data records, and mechanisms for synchronizing repositories across the network. Repositories are cryptographically verifiable and publicly published. Key goals for the working group:

- A low-latency streaming synchronization protocol, including notification of data deletion
- The streaming protocol should allow for alternative network transports
- Scale to global network applications
- Support for redistribution of authenticated data via multiple layers of intermediary parties
- Encoding format for structured data records (CBOR and JSON representations)
- Export format for transfer of data repositories
- URI scheme for persistent references between data records
- Account migration between network locations (eg, hosting providers) does not break references or graph relationships

Verification of signatures requires an account identity system with public key management. The output of this working group will not mandate any single identity system implementation. It will instead describe an interface boundary and requirements for identity systems to fulfill that interface. One key requirement will be support for reliable migration of account data repositories between hosting providers.

The following are out of scope for the working group:

- Development or standardization of specific network account identity systems
- Transfer of non-public or limited-visibility data
- Any application-specific data schemas
- Any social application semantics

The ATP working group will consult the Crypto Forum Research Group (CFRG) and Security Area groups concerning the use and selection of any cryptographic systems. Privacy and Centralization considerations will be documented (eg, per RFC 6873) and will inform design decisions.

