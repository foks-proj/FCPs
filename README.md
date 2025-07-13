
Forked from: [Avalanche Community Proposals (ACP)](https://github.com/avalanche-foundation/ACPs)

## What is a FOKS Community Proposal (FCP)?

A FOKS Community Proposal is a concise document that introduces a change or best practice for adoption on the [Federated Open Key Service Network](https://foks.pub). FCPs should provide clear technical specifications of any proposals and a compelling rationale for their adoption.

FCPs are an open framework for proposing improvements and gathering consensus
around changes to the Federated Open Key Service Network. FCPs can be proposed
by anyone and will be merged into this repository as long as they are
well-formatted and coherent. Once enough support is gathered from the community,
it may be scheduled for activation on the Federated Open Key Service Network by
FOKS clients and servers.

## FCP Tracks

There are four kinds of FCP:

* A `Standards Track` FCP describes a change to the design or function of the FOKS Network, such as a changes to users, teams, hostchains, or adding new features.
* A `Meta Track` FCP describes a change to the FCP process or suggests a new way for the FOKS Community to collaborate.

## FCP Statuses

There are four statuses of an FCP:

* A `Proposed` FCP has been merged into the main branch of the FCP repository. It is actively being discussed by the FOKS Community and may be modified based on feedback.
* An `Implementable` FCP is considered "ready for implementation" by the author(s) and will no longer change meaningfully from its current form (which would require a new FCP).
* An `Activated` FCP has been activated on the FOKS Network via a coordinated upgrade by the FOKS Community. Once an FCP is `Activated`, it is locked.
* A `Stale` FCP has been abandoned by its author(s) because it is not supported by the FOKS Community or has been replaced with another FCP.

## FCP Workflow

### Step 0: Think of a Novel Improvement to FOKS

The FCP process begins with a new idea for FOKS. Each potential FCP must have an author(s): someone who writes the FCP using the style and format described below, shepherds the associated GitHub Discussion, and attempts to build consensus around the idea. Note that ideas and any resulting FCP is public. Authors should not post any ideas or anything in an FCP that the Author wants to keep confidential or to keep ownership rights in (such as intellectual property rights).

### Step 1: Post Your Idea to [GitHub Discussions](https://github.com/foks-proj/FCPs/discussions/categories/ideas)

The author(s) should first attempt to ascertain whether there is support for their idea by posting in the "Ideas" category of GitHub Discussions. Vetting an idea publicly before going as far as writing an FCP is meant to save both the potential author(s) and the wider FOKS Community time. Asking the FOKS Community first if an idea is original helps prevent too much time being spent on something that is guaranteed to be rejected based on prior discussions (searching the Internet does not always do the trick). It also helps to make sure the idea is applicable to the entire community and not just the author(s). Small enhancements or patches often don't need standardization between multiple projects; these don't need an FCP and should be injected into the relevant development workflow with a patch submission to the applicable issue tracker.

### Step 2: Propose an FCP via [Pull Request](https://github.com/foks-proj/FCPs/pulls)

Once the author(s) feels confident that an idea has a decent chance of acceptance, an FCP should be drafted and submitted as a pull request (PR). This draft must be written in FCP style as described below. It is highly recommended that a single FCP contain a single key proposal or new idea. The more focused the FCP, the more successful it tends to be. If in doubt, split your FCP into several well-focused ones. The PR number of the FCP will become its assigned number.

### Step 3: Build Consensus on [GitHub Discussions](https://github.com/foks-proj/FCPs/discussions/categories/discussion) and Provide an Implementation (if Applicable)

FCPs will be merged by FCP maintainers if the proposal is generally well-formatted and coherent. FCP editors will attempt to merge anything worthy of discussion, regardless of feasibility or complexity, that is not a duplicate or incomplete. After an FCP is merged, an official GitHub Discussion will be opened for the FCP and linked to the proposal for community discussion. It is recommended for author(s) or supportive FOKS Community members to post an accompanying non-technical overview of their FCP for general consumption in this GitHub Discussion. The FCP should be reviewed and broadly supported before a reference implementation is started, again to avoid wasting the author(s) and the FOKS Community's time, unless a reference implementation will aid people in studying the FCP.

### Step 4: Mark FCP as `Implementable` via [Pull Request](https://github.com/foks-proj/FCPs/pulls)

Once an FCP is considered complete by the author(s), it should be marked as
`Implementable`. At this point, all open questions should be addressed and an
associated reference implementation should be provided (if applicable). 

### [Optional] Step 5: Mark FCP as `Stale` via [Pull Request](https://github.com/foks-proj/FCPs/pulls)

An FCP can be superseded by a different FCP, rendering the original obsolete. If this occurs, the original FCP will be marked as `Stale`. FCPs may also be marked as `Stale` if the author(s) abandon work on it for a prolonged period of time (12+ months). FCPs may be reopened and moved back to `Proposed` if the author(s) restart work.

### Maintenance

FCP maintainers will only merge PRs updating an FCP if it is created or approved by at least one of the author(s). FCP maintainers are not responsible for ensuring FCP author(s) approve the PR. FCP author(s) are expected to review PRs that target their unlocked FCP (`Proposed` or `Implementable`). Any PRs opened against a locked FCP (`Activated` or `Stale`) will not be merged by FCP maintainers.

## What belongs in a successful FCP?

Each FCP must have the following parts:

* `Preamble`: Markdown table containing metadata about the FCP, including the FCP number, a short descriptive title, the author(s), and optionally the contact info for each author, etc.
* `Abstract`: Concise (~200 word) description of the FCP
* `Motivation`: Rationale for adopting the FCP and the specific issue/challenge/opportunity it addresses
* `Specification`: Complete description of the semantics of any change should allow any Community member to implement the FCP
* `Security Considerations`: Security implications of the proposed FCP

Each FCP can have the following parts:

* `Open Questions`: Questions that should be resolved before implementation

Each `Standards Track` FCP must have the following parts:

* `Backwards Compatibility`: List of backwards incompatible changes required to implement the FCP and their impact on the FOKS Network
* `Reference Implementation`: Code, documentation, and telemetry (from a local network) of the FCP change

### FCP Formats and Templates

Each FCP is allocated a unique subdirectory in the `FCPs` directory. The name of this subdirectory must be of the form `N-T` where `N` is the FCP number and `T` is the FCP title with any spaces replaced by hyphens. FCPs must be written in [markdown](https://daringfireball.net/projects/markdown/syntax) format and stored at `FCPs/N-T/README.md`. Please see the [FCP template](./FCPs/TEMPLATE.md) for an example of the correct layout.

### Auxiliary Files

FCPs may include auxiliary files such as diagrams or code snippets. Such files should be stored in the FCP's subdirectory (`FCPs/N-T/*`). There is no required naming convention for auxiliary files.

### Waived Copyright

FCP authors must waive any copyright claims before an FCP will be merged into the repository. This can be done by including the following text in an FCP:

```text
## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
```

## Proposals

## Contributing

Before contributing to FCPs, please read the [FCP Terms of Contribution](./CONTRIBUTING.md).
