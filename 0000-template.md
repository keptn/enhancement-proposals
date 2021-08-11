# Role-based Access Control (RBAC)

**Success Criteria:** Role-based and SSO-based access control need to be applied for accessing Keptn UIs and APIs.

## Motivation

**Target audience pain points**
As any user entering the Bridge, I see all and everything there from my whole organization. I see things, I do not care about and even worse: I might also see things I should not be allowed to see (e.g. QG results including security vulnerability metrics for a certain release).

**Evidence**
Keptn users requested role-based access to the Bridge several times.

No discussion, that role-based access and visibility of company data is a must-have for any enterprise-ready solution.

## What

**What it is? What it is not?**
* As anyone looking into data in the Bridge, I should only authenticate once and have proper permissions within the Bridge based on my SSO roles.

* As an automation engineer, I should only edit the automation I'm responsible for.

* As a member of one department, I might not be allowed to see (and not allowed to edit) quality gates of another department.

## Use Cases

**Keptn has three roles: read / write / admin**

* A read role allows you to:
  * Read all resources without enabling you to modify a resource

* A write role allows you to:
  * Modify all resources. E.g. approve a promotion in the bridge
  * The "write" role for a given stage should only permit any modifications within the same stage. E.g. if you have the "write" role in dev, but not in hardening it should not allow you to press the "promote" button in dev, since then modifications are done in hardening.

* An admin role allows you to:
  * Modify all resources, change the Keptn shipyard, create projects, ...

**Roles in Keptn must be settable on project- / service- / (stage+service)- level**

* Having a role on project-level enables you to perform an action in each stage for each service
  * E.g. when having the write role for service X you can approve any approval step for service X in any stage
  * Example use cases: define admins per project / give read permissions to business stakeholders

* Having a role on service-level enables you to perform an action only for the given service (in each stage)
  * E.g. when having the write role for service X you can approve any approval step for service X in any stage
  * Example use case: All members of a team can promote a service to the next stage

* Having a role on (stage+service)-level enables you to perform an action only for the given stage/service tuple
  * E.g. when having the write role for service X in stage Y you can only approve approval steps for service X in stage Y
  * Example use case: Only a handful of people can approve the promotion of a service to production

**A role has one or more API tokens**

**The API tokens can be rotated**

* That's why we need multiple API tokens per role: We want to prevent that access is lost during the time the API token is rotated
