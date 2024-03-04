# Releasing Stable Version

Releasing a stable version of Keptn under the v2.0.0 tag and bump the API to v1.

**State: ACCEPTED**

## Motivation

Keptn reached feature parity with the old Keptn v1.
The codebase is mature enough to warrant a stable release.

## Trade-offs and mitigations

The APIs defined via HTTPs and CRDs are mature enough to cover all the use cases that Keptn wants to cover.
We don't expect the existing CRDs and REST endpoints to change.


## Breaking changes

In case of new use cases, we can look into introducing conversion webhooks with sensible defaults to have non breaking behaviors.
Otherwise, new CRDs can be created extending the Keptn API surface in a non breaking manner.

