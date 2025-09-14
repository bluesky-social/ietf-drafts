
# Authenticated Transfer (AT) IETF Drafts

This git repository contains working text for internet drafts (I-Ds) submitted to the IETF related to Authenticated Transfer (AT). AT is a collection of protocol components that together provide a generic framework for interoperable social web applications, using global aggregations of interlinked, self-certifying data records. You can read more at [atproto.com](https://atproto.com).

Authenticated Transfer is not currently an IETF standard and there is not yet an IETF Working Group for AT. A [BoF request](https://datatracker.ietf.org/doc/bofreq-newbold-authenticated-transfer/) has been submitted for IETF 125 meeting (November 2025 in Montreal).

Until then, these drafts, and this repository, are not an active venue for the AT developer ecosystem ("Atmosphere"). Most protocol-level discussion takes place in the [atproto Github discussions](https://github.com/bluesky-social/atproto/discussions) or [atproto specifications repository](https://github.com/bluesky-social/atproto-website).

However, editorial feedback and clarifying questions on the draft text is welcome, and can be submitted here as issues or pull-requests.

As these documents fall under the broader IETF community, please be aware of IETF policies around participation as outlined in [CONTRIBUTING](./CONTRIBUTING.md).

## Document Tooling

This repository uses the excellent [I-D template system](https://github.com/martinthomson/i-d-template) developed by Martin Thomson. 

Formatted text and HTML versions of the draft can be built using `make`.

```sh
$ make
```

Command line usage requires that you have the necessary software installed.  See [the instructions](https://github.com/martinthomson/i-d-template/blob/main/doc/SETUP.md).
