# Replace B/G deployment support of Helm-service with Argo Rollouts

This KEP proposes to use Argo Rollouts for blue/green deployments instead of letting the Helm-service generate a second deployment and service.

## Motivation

For blue/green deployments, the Helm-service generates a second chart, which contains the primary deployment (i.e. stable version) and corresponding networking resources (i.e. VirtualServices and DesitnationRules).
This generation only works under the following condistions and comes with the following drawbacks:
- The Helm chart provided by the user must contain exactly one deployment and one service, which prevents to deploy multiple services together (i.e. release brackets)
- All services are automatically exposed via a VirtualService, i.e. Keptn cannot decide whether a service should be exposed or not
- Istio is required
- Does not fully support edits in GIT, e.g. regenerates VirtualServices when an artifact is promoted
- Does not allow to route traffic other than HTTP
- Does not allow to rollback to an old version, i.e. the Helm-service executes multiple Git-commits and Helm upgrades for each deployment
- Does not allow to have different Helm charts per stage and service. This is a requirement in almost every production setting (i.e. dev has a different config than production).

Overall, these limits the B/G capabilities of the Helm-service.

This KEP proposes to completely remove the generation of the generated-chart in the Helm-service and
instead use [Argo Rollouts](https://argoproj.github.io/argo-rollouts/) for blue/green deployments as well as canary releases.

The new responsibilities of the Helm-service would be :
- Deployment: Execute a Helm upgrade with `helm upgrade KEPTN_SERVICE_NAME CHART -n KEPTN_PROJECT-KEPTN_STAGE --create-namespace --install`

Benefits by implementing this KEP:
- Support of Keptn service which consists of multiple deployments, services, etc. (i.e. release brackets)
- User can decide if a service is exposed or not
- Argo Rollouts provide out-of-the-box Canary support
- Direct and b/g deploymentes can be mixed within a project
- Removes Istio dependency
- Easy/out-of-the-box rollback
- Configuration changes are done via Git edits
- Work with PRs, etc. 

## Explanation

<!-- keptn create project sockshop --shipyard=~/Desktop/examples/onboarding-carts/shipyard.yaml 
keptn create service carts --project=sockshop
keptn add-resource  --project=sockshop --stage=dev --service=carts --resource=carts-0.1.0.tgz --resourceUri=helm/carts.tgz

```
{
  "contenttype": "application/json",
  "data": {
    "project": "sockshop",
    "service": "carts",
    "stage": "dev"    
  },
  "id": "9f2e277f-3144-4941-baea-712a56d59857",
  "source": "https://github.com/keptn/keptn/cli#deployment.triggered",
  "specversion": "0.2",
  "time": "2020-07-28T14:47:12.797Z",
  "type": "sh.keptn.event.deployment.triggered"
}
```

keptn send event -f ~/Desktop/event.json  -->


## Internal details



## Trade-offs and mitigations

* Keptn does not allow to do blue/green deployments without an external dependency, i.e. Argo-rollout.

* Backwards compatibility: Services onboarded with the Helm-service consist of a user- and generated-chart. These charts can only be maintained by the helm-service. Therefore, in oder to remain backwards-compatible in 0.8.0
    * we still need the current version of the Helm-service,
    * the event flow defined in the shipyard is then triggered by the `deployment-finished` event sent out by the Helm-service,
    * the gatekeeper still has to send a configuration.change event for promoting artifacts
* We need to stop the support of the Helm-service with Keptn 1.0


## Breaking changes

- We cannot define the deployment strategy in the shipyard anymore. Instead this is defined in the Helm-chart.
- User have to define an Argo rollout resource for doing a B/G deployment. This possibly adds complexity but gives great flexibility.
- Users have to onboard a service for all stages when different deployment strategies are used. But this is required anyway, if Helm charts are different.

## Open questions

- Does Dynatrace allow to retrieve metrics for the different versions in a Rollout? Prometheus works.
- If charts are different in the respective stages, we cannot support the automatic promotion of an artifact in the next stage because we cannot simply apply the same configuration change for the next stage.
- If the Helm chart contains multiple services which are deployed, which service should then get tested and evaluated? This destroys our mapping between a Keptn service and a k8s deployment.

## Future possibilities

## Production 
- Amasol, Avodaq