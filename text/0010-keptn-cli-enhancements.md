# keptn cli and API enhancements

Enhance keptn-cli with the ability to list and get details of onboarded projects and services.

## Motivation

Right now when I'm using keptn within a CI/CD Pipeline I want to check if my service is already onboarded in keptn, if not it should be done automatically.
From Ops-Perspective I want to get a list of all projects which are using keptn. Like in helm a keptn list command to retrieve a list of all projects and for more details a keptn get/details <project> command which returns all services within the project would be benificial.
As workaround I'm checking if in my kubernetes cluster a lighthouse-configmap is existing, but this is not a pretty solution.

## Explanation

```bash
# keptn list
Project             Onboarded                       Git Repository
sockshop            2020-02-03 11:50                https://xyz/project.git
```

```bash
# keptn get sockshop
Project: sockshop
Onboarded: 2020-02-03 11:50
Git Repository: https://xyz/project.git
Services:
- carts-service
- payment-service
```

```bash
# keptn get sockshop service carts-service
Project: sockshop
Service: carts-service
Last artifact: myimage:v1.0.2
Last keptn event: sh.keptn.events.evaluation-done
Stage
Last deployment stage: dev
Last deployment time: 2020-02-03 11:50
```

## Internal details

A keptn API Endpoint which the cli could use should do the job, and instead of using the cli within a CI/CD Job, a simple curl command could be used to get the information.

## Trade-offs and mitigations

This is a keptn cli enhancement and should not have any drawbacks.

## Breaking changes

Could this proposal cause any breaking changes (e.g., in the spec, within any workflows)?

## Prior art and alternatives

As described in the Motivation-section, I'm checking if a lighthose config map exists within the k8s cluster. But this is only a workaround.

## Open questions

N/A

## Future possibilities

N/A