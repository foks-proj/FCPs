| FCP | PR Number |
| :--- | :--- |
| **Title** | FCP title |
| **Author(s)** | A list of the author's name(s) and optionally contact info: FirstName LastName ([@GitHubUsername](https://github.com/Username) or [email@address.com](mailto:email@address.com)) |
| **Status** | Proposed, Implementable, Activated, Stale ([Discussion](POPULATED BY MAINTAINER, DO NOT SET)) |
| **Track** | Standards, Best Practices, Meta, Subnet |
| **Replaces (\*optional)** | [FCP-XX](../XXX/README.md) |
| **Superseded-By (\*optional)** | [FCP-XX](../XXX/README.md) |

**Note on the preamble:** The `Discussion` link is only needed for FCPs in `Proposed` and `Implementable` status.

This is the suggested template for new FCPs.

## Abstract

A concise (~200 word) description of the FCP being proposed. This should be a very clear and human-readable version of the specification section. Someone should be able to read only the abstract to get the gist of what this FCP is about.

## Motivation

The motivation is critical for FCPs that want to change the functionality of the FOKS Network. It should clearly explain why the existing FOKS Network implementation is inadequate to address the problem that the FCP solves.

## Specification

The technical specification should describe the syntax and semantics of any FCP. The specification should be detailed enough to allow any FOKS client or server author to implement the FCP without consultation from the author(s).

## Backwards Compatibility

All FCPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The FCP should provide recommendations for dealing with these incompatibilities.

## Reference Implementation

A reference/example implementation that people can use to assist in understanding or implementing this specification. If the implementation is too large to reasonably be included inline, then consider adding it to the FCP subdirectory or linking to a PR in an external repository.

## Needed Tests

All major changes must be accompanied by tests that verify correct operation of the feature,
backwards- and forwards-compatibility behavior, and appropriate handling of forseen attacks
on cryptographic assurances.

## CLI Changes

If the FCP introduces changes that will be user-facing, there should be a discussion of what
changes are needed to the FOKS CLI.

## Security Considerations

Optional section that discusses the security implications/considerations relevant to the proposed change.

## Open Questions

Optional section that lists any concerns that should be resolved prior to implementation.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
