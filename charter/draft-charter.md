Authenticated Transfer Protocol (ATP) Working Group
===================================================

The Authenticated Transfer Protocol (ATP) working group will develop standards-track specifications for self-certifying data structures and network protocols which enable efficient synchronization of public data across organizational boundaries, while preserving authenticity.

The work will be based on draft-holmgren-at-repository and the pre-standardization implementation and deployment experience of multiple parties in the ATP ecosystem. All existing work must undergo a consensus process, and changes may be necessary, but gratuitous changes will be avoided when possible.

The working group will focus on a general purpose data repository structure for public user data records, and mechanisms for synchronizing repositories across the network. Repositories are cryptographically verifiable and publicly published. Key goals for the working group:

- A format for full repository import/export
- A low-latency streaming synchronization protocol, including notification of data deletion
- The streaming protocol should allow for alternative network transports
- Support for redistribution of authenticated data via multiple layers of intermediary parties
- A URI scheme for references between data records.

Verification of signatures requires an account identity system with public key management. The output of this working group will not mandate any single identity system implementation. It will instead describe an interface boundary and requirements for identity systems to fulfill that interface. One key requirement will be support for reliable migration of account data repositories between hosting providers.

The following are out of scope for the working group:

- Development or standardization of specific network account identity systems
- Transfer of non-public or limited-visibility data
- Any application-specific data schemas
- Any social application semantics

The ATP working group will consult the Crypto Forum Research Group (CFRG) and Security Area groups concerning the use and selection of any cryptographic systems. Privacy and Centralization considerations will be documented (eg, per RFC 6873) and will inform design decisions.

