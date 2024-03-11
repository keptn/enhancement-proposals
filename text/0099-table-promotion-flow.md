# Table Promotion Flow

**State: DRAFTING**

Allow starting deployment flows from previous artifacts which are perhaps multiple iterations previous.

## Motivation

Keptn sits in a very interesting place in the CI/CD CNCF space, where it helps by gluing many tools together
which otherwise may have a gap. One of these gaps is visibility on environment management and deployment management
with defined sources of artifacts and defined possible environments.

## Explanation

This table would be created by defining artifact sources, sources of deployment information (ex. ArgoCD), and yaml which 
defines the possible environments and how promotion happens. Maybe this could be in Shipyard

A UI ties all this together, with a table where rows are artifacts (or sets of artifacts), and columns are environments.
You should be able to click a button to trigger either a rollback to any previous artifact, or if quality gates are green
then promote forward into a downstream environment.

### Prerequisites / Proposed solution

* Source of artifacts with query to represent which ones are relevant for this project
* Sources of deployments with information to relate with the artifacts to identify what is currently deployed in what
environment

## Open questions/Implementation details

* How to store the deployment information statelessly? ArgoCD can be queried, but this is a big assumption that ArgoCD
is being used
* Deployment needs to be done indirectly, ex. by triggering an automation tool which modifies a GitOps repo. This somehow
needs to be setup or else the functionality will violate GitOps
