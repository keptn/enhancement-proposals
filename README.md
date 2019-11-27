# Keptn Enhancement Proposals (KEP)

This repository contains **K**eptn **E**nhancement **P**roposals (one could also call these Request For Comments). A new proposal can be made by creating a pull request. Already accepted proposals will be listed in the [text/](text/) directory.

## What can you find in this repo

All enhancement proposals that have been accepted into this repo can be found in the [text/](text/) directory in the following format:

`XXXX-my-awesome-proposal.md` where as `XXXX` is the KEP ID (basically the ID of the Pull Request).

Each proposal can have one of the following status:

* proposed (PR created)
* approved (PR reviewed and merged)
* rejected (PR is actively rejected by the reviewers)
* withdrawn (PR has been closed by the proposer)

## What changes require a KEP?

A KEP is required when it is intended to **introduce new behaviour**, **change desired behaviour**, or otherwise **modify requirements** of Keptn.

In practice, this means that KEPs should be used for such changes as:

* Behavioural changes of Keptn or any core services
* Changes that affect the interaction of multiple services
* Breaking changes

On the other hand, they do not necessarily need to be used for such changes as:

* Bug fixes
* Rephrasing, grammatical fixes, typos, etc.
* Refactoring
* Automated tests
* Simple workflow changes (such as adding a timeout, retry logic, ...)

**Note:** The above lists are intended only as examples and are not meant to be exhaustive. If you don't know whether a change requires a KEP, please [feel free to contact us](https://github.com/keptn/community)!

## Writing a new proposal

1. [Fork](https://help.github.com/en/articles/fork-a-repo) the `keptn/enhancement-proposals` repo
1. Copy `0000-template.md` into the [text/](text/) directory and rename it accordingly (e.g., to `XXXX-my-proposal-title.md`) - Please note that `XXXX` (ID of the KEP) needs to be replaced with the Pull Request ID later.
1. Fill in the template. Put care into the details: It is important to present convincing motivation, demonstrate an understanding of the design's impact, and honestly assess the drawbacks and potential alternatives (feel free to adapt the template to your likings, if you feel that it is necessary or if it helps to improve readability).

## Submitting a new proposal

* A KEP is `proposed` by posting it as a Pull Request (PR). Once the PR is created, update the KEP file name to use the PR ID as the KEP ID.
* A KEP is `approved` after a number of [official reviewers](CODEOWNERS) github-approve the PR (this number might change over time). The RFC is then merged.
* If a KEP is `rejected` or `withdrawn`, the PR is closed. Note that these KEP submissions are still recorded, as Github retains both the discussion and the proposal, even if the branch is later deleted.
* If a KEP discussion becomes long, and the KEP then goes through a major revision, the next version of the KEP can be posted as a new PR, which references the old PR. The old PR is then closed. This makes KEP review easier to follow and participate in.

# Final Words

This process borrows from [Open Telemetry Enhancement Proposals](https://github.com/open-telemetry/oteps).

The hope and expectation is that this process will evolve over time, it is by no means fixed.
If you have any suggestions, questions or concerns, please [get in touch with us](https://github.com/keptn/community).

