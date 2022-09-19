# Keptn LTS Release Line and Keptn 1.0


## Motivation

It is a common practice for the CNCF projects to offer a Long-Term Support (LTS) release line.
Such a release line is important for enterprise users, and it could help Keptn adoption.


## Explanation

The proposal is to introduce a new LTS baseline
which would be based on the current Keptn version of the Keptn core and key services.

This KEP suggests the following:

* Introducing a first ever LTS release line in Keptn, versioned as Keptn 1.0
* List of components that would be managed as a part of the LTS
* Scope and timeline for the 1.0 release
* Release policy for versions beyond 1.0
* Changes in the release policy for the common release baseline


## Internal details


### Scope of the LTS

Keptn LTS scope would include not just the Keptn core components and CLI,
but also key services popular among Keptn users and considered critical for the ecosystem.
Also, Keptn specification would be pinned.


#### Keptn Core components

* Keptn core
* Keptn Bridge
* Keptn CLI
* Installation and packaging scripts
* Helm charts


#### Keptn Specification

Keptn specification would be pinned for the LTS. There will be no breaking changes in the specification within the LTS release, and minimum changes at all. The specification documentation may be extended over time for quality reasons.


#### Keptn services to be included into the LTS

Keptn LTS will include a number of critical services but not all of them.
The proposed list:

* All services within [https://github.com/keptn/keptn](https://github.com/keptn/keptn) 
* Webhook Service
* Prometheus service
* Job Executor Service (JES)
* Dynatrace Service (Confirmed with Dynatrace team maintaining it)
* TBD - Datadog service
    * Note: Currently the service is in the sandbox
* TBD - Helm service - most likely NO
* TBD - JMeter service - most likely NO
    * Recommendation: Use Job Executor service
* TBD - Unleash service - for demoing the auto-remediation use-cases
    * ON: We can use webhook service for auto-remediation demos. Or the Job executor service

The list might be extended overtime.
All included services have to be actively maintained, 
have a good record of quality and
should be promoted to [github.com/keptn-contrib](https://github.com/keptn-contrib) before they are considered for inclusion.
We also expect the best practices (e.g. using Helm charts) to be followed.

All Keptn services included into the LTS will have a badge indicating the service is included into Keptn LTS. 

Out of the scope:

* Changes in the Maintenance policy.
  All services will be maintained as is by their maintainers.
  Being included into LTS does not reserve special rights for Keptn maintainers, e.g. on-demand security releases.
    * It’s to be handled separately by the community
* Adoption policy for services
    * In this KEP we do not define a policy for services that are no longer maintained but included into the LTS 


### Compatibility policy

Once the user upgrades to the LTS baseline, they should be confident there will be no breaking changes
UNLESS there are security issues or critical defects that require a breaking change.
The specification would be also pinned and extended only in exceptional cases;
there should be no breaking changes in it between the versions.

### Versioning scheme


#### Keptn core versioning

Semantic versioning should be used for the LTS baselines: [https://semver.org/](https://semver.org/).
Given a version number MAJOR.MINOR.PATCH, increment the:

1. MAJOR version when you make incompatible API changes
2. MINOR version when you add functionality in a backwards compatible manner
3. PATCH version when you make backwards compatible bug fixes

The first version of Keptn LTS will be 1.0.0,
the PATCH version will be used for incremental releases of this baseline.
For future baselines, first two digits will be used as a base Keptn version selected for LTS,
with the new LTS release with backports being effectively MAJOR.MINOR.1.

Keptn services and the specification would follow the similar release scheme, with releases on-demand.


### Support timeline and policy


* Keptn LTS versions will be supported for at least 6 months after the initial release.
  The Keptn community decides when to stop the support
    * Keptn contributors and users can be extended beyond 6 months, as long as community members (maintainers, end users or vendors) are willing to provide support.
    * 6 months timeline is chosen initially to evaluate LTS and its adoption. The minimum support term may be extended later based on the evaluation
* Keptn community announces the LTS baseline end-of-life with 3 months advance, 
  using the community announcement channels
 (#announcements on Slack is considered the official one)
* If Keptn LTS includes preview features, they are not covered by the LTS support policy.
  Breaking changes might be introduced there even within LTS,
  and the preview features can be also removed altogether should it be needed for better LTS stability.

### Release cadence 

We are not sure about the adoption of the LTS release and the cadence of the upcoming major changes in Keptn components.
Based on that, the approach in this KEP makes no commitment on release cadence:

* Keptn community may release new LTS baselines based on the decision by the community.
  Any community member can suggest making a new LTS release baseline.
* For patch releases, the Keptn community intends releasing a new version with backports
  when there are major bug fixes that need to be backported.
  Any community member can suggest making a new LTS patch release, and the decision will be made by Keptn community and core maintainers
* For security issues, the [Keptn Security Process](https://github.com/keptn/.github/blob/main/SECURITY.md) will be followed for all currently supported LTS baselines.
  Security patches for the LTS need to be released along with security patches for the main baseline.

For the normal release line, we also remove monthly releases for MINOR releases with new features.
We target releasing Keptn on a monthly basis if there is something to deliver.
Patch releases for bugfixes and security releases will be done on-demand. 


## Changes to be made

### Keptn Specification

No changes planned at the moment, except documentation.

### Keptn Core

Keptn core changes to be minimized to the pending stories:

* Performance improvements
* Code cleanup
* Maintainability & Supportability
* Landing open pull requests with valuable functionality

Notably Keptn GitOps will remain in preview, or KEP-60 RBAC will not be shipped in the first LTS release.
The timeline for these releases is likely to be delayed by the community.


### Documentation changes

In the LTS we want to improve maintainability of documentation and
to land pending stories in the documentation working group.


* Adopt Documentation-as-Code in Keptn where reasonable
    * Support for distributed documentation: multiple repositories, branches and versions.
    [GSoC 2022 project](https://github.com/keptn/community/blob/main/mentorship/gsoc/2022/projects/new-docs-site-engine/README.md)
    * We keep documentation source in the production codebase to the extent that this does not degrade the user experience  (e.g. project and service repositories)
    * Solve the version sprawl in Keptn Docs
* Tutorials 2.0
    * Interactive tutorials. Continue Killercoda adoption
    * Revamp the tutorials site, archive the existing tutorials
    * Update tutorials for Keptn 1.0 LTS
    * TBD - Prometheus, ArgoCD, [TBD for Dynatrace and Datadog]
* TBD - Developer Guide 1.0
    * We just group relevant documents from other sections

## Trade-offs and mitigations

* Time-to-market for new features will be longer for users of LTS releases.
  Preview features can be used as mitigations.


## Breaking changes

There are no breaking changes coming out as a part of this proposal.
At the same time, the LTS release would integrate breaking changes released in the previous Keptn versions.
Users will have to follow the upgrade guidelines to upgrade to the LTS.
The guidelines will remain as is (from one minor version to another).

Once the user upgrades to the LTS baseline,
they should be confident there will be no breaking changes
UNLESS there are security issues or critical defects that require a breaking change.


## Prior art and alternatives

The proposal is based on the experience of LTS baselines in the CNCF projects and in Jenkins LTS baseline.


## Open questions

* What is the scope for Keptn Services within the LTS? See the proposal above
* Will we attempt to test/upgrade integrations in keptn-contrib before releasing V1?  
* Will we attempt to identify critical videos that need to be reshot for V1 or marked as accurate for V1?
* Should we introduce a new maintenance and adoption policy to ensure continuity of maintenance for services included into LTS?


## Future possibilities

This change enables introducing new LTS baselines in the community on-demand when the community believes there are important features to be shipped.


## References

* Semantic versioning 2.0: [https://semver.org/](https://semver.org/) 
* LTS Model in Jenkins: [https://www.jenkins.io/download/lts/](https://www.jenkins.io/download/lts/) 
