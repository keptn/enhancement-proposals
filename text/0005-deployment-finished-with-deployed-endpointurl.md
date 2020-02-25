# Enhance Deployment Finished Event With Deployed Endpoint URL

Deployment Tool adds endpoint of deployed service to Deployment Finished Event!

## Motivation

Right now the Keptn Services that are handling Test Execution by reacting to the Deployment Finished events have to GUESS which endpoint they actually have to test. Right now e.g: JMeter Services assumes the service is accessible via http://project.service.stage.deploymenttype.
We have to take this guesswork out of the equation by letting the Deployment Tool also pass the Deployed Endpoint as part of the Cloud Native Event. This not only eliminates the "guesswork" we assume Keptn Services do, it also allows us to better integrate Keptn with external tools that cover the deployment phase, e.g: Jenkins, ArgoCD, GitLab Pipelines ...

## Explanation

The tool or service that actually deploys an artifact is the tool/service that knows best which endpoint the deployed service is reachable. This could either be a Keptn Service that reacts to a Configuration Change Event or it could be an external deployment pipeline such as ArgoCD, Jenkins, ...

As these tools are currently sending the Deployment Finished Event to Keptn I propose that these tools add and additional field called "DeployedEndpoint". This field should contain the endpoint that can be reached by other tools such as a testing tool, synthethic or health check ..., e.g: https://myservice.staging.mycluster.local or https://myjavaapp.mycompany.com

## Internal details

I propose that we extend the Deployment Finished Event with two new fields called "deploymentURILocal" and "deploymentURIPublic". Deployment Tools that send a deployment finished event to Keptn SHOULD (but are not obliged) to fill these fields.
* If the "deploymentURILocal" field IS PROVIDED, Keptn services such as the JMeter, Neotys, etc. can use this URL to execute their tests against.
* If the "deploymentURILocal" field IS NOT PROVIDED, but the field "deploymentURIPublic" is provided, Keptn services such as the JMeter, Neotys, etc. can use this URL to execute their tests against.
* If non of these fields are provided, services such as JMeter, Neotys, etc. can fall back to their current "guestimation" of the URL.

## Trade-offs and mitigations

No tradeoffs as it is backward compatible. The deployment tools simply add more context data which helps other tools later in the workflow to do a better job!

## Breaking changes

No breaking changes as we just add a new optional field.

## Prior art and alternatives

Not aware of any alternatives other than our current "guestimage" approach

## Open questions

How shall we name that field?
Is one field enough or shall we provide a set of fields, e.g: root endpoint, health check endpoint ...?

## Future possibilities

This enables a true "Performance as a Self-Service" implementation by having existing pipelines deploy applications in any type of enviornment and then have Keptn take care of automated testing and test evaluation!
