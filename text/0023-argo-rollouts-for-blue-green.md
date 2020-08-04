# Replace B/G deployment support of Helm-service with Argo Rollouts

This KEP proposes to use Argo Rollouts for blue/green deployments instead of letting the helm-service generate a second deployment and service.

## Motivation

**Status quo:**
For blue/green deployments, the Helm-service generates a second chart, which contains the primary deployment (i.e. stable version) and corresponding networking resources (i.e. VirtualServices and DesitnationRules).

This generation only works under the following conditions: 
- All services are automatically exposed via a VirtualService, i.e. Keptn cannot decide whether a service should be exposed or not
- Istio is required

**Problem statement:** The resulting drawbacks are: 
- As the Helm chart provided by the user can only contain one deployment and one service, it is impossible to deploy multiple services together (i.e. release brackets)
- Does not fully support edits in Git, e.g. regenerates VirtualServices when an artifact is promoted
- Does not allow to route traffic other than HTTP
- Does not allow to rollback to an old version, i.e. the Helm-service executes multiple Git-commits and Helm upgrades for each deployment
- Does not allow to have different Helm charts per stage and service. This is a requirement in almost every production setting (i.e. dev has a different config than production).

Overall, these drawbacks limit the blue/green deployment/releasing capabilities of the Helm-service.

**Proposal:** This KEP proposes to completely remove the generation of the `generated-chart` in the helm-service and instead use [Argo Rollouts](https://argoproj.github.io/argo-rollouts/) for blue/green deployments as well as canary releases.

The new responsibilities of the Helm-service would be:
- *Deployment*: Execute a Helm upgrade with `helm upgrade KEPTN_SERVICE_NAME CHART -n KEPTN_PROJECT-KEPTN_STAGE --create-namespace --install`

**Benefits:** By implementing this KEP, the following advantages can be achieved:
- Support of Keptn service, which consists of multiple deployments, services, etc. (i.e. release brackets)
- User can decide if a service is exposed or not
- Argo Rollouts provide out-of-the-box canary support
- Direct and blue/green deployments can be mixed within a project
- Removes Istio dependency
- Easy/out-of-the-box rollback
- Configuration changes are done via Git edits
- GitOps approach: Work with PRs to sync stages 

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

* Keptn does not allow to do blue/green deployments without an external dependency, i.e. Argo Rollouts.

* Backward compatibility: Services onboarded with the helm-service consist of a user- and generated-chart. These charts can only be maintained by the helm-service. Therefore, in order to remain backward-compatible in 0.8.0:
    * The current version of the helm-service is still needed,
    * The event flow defined in the shipyard is then triggered by the `deployment-finished` event sent out by the helm-service,
    * The gatekeeper still has to send a configuration.change event for promoting artifacts
* The support of the helm-service will be stopped with Keptn 1.0


## Breaking changes

- The deployment strategy cannot be defined in the shipyard anymore. Instead, this is defined in the Helm-chart.
- Users have to define an Argo rollout resource for doing a B/G deployment. This possibly adds complexity on the one hand but gives flexibility on the other hand.
- Users have to onboard a service for all stages when different deployment strategies are used. But this is required anyway if Helm charts are different.

## Open questions

- Does Dynatrace allow to retrieve metrics for the different versions in a Rollout? Prometheus works.
- If charts are different in the respective stages, we cannot support the automatic promotion of an artifact in the next stage because we cannot simply apply the same configuration change for the next stage.
- If the Helm chart contains multiple services that are deployed, which service should then get tested and evaluated? This destroys our mapping between a Keptn service and a K8s deployment.

## Future possibilities
