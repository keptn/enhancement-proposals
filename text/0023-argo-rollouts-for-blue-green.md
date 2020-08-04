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
- Allows to deploy a temporary branch (i.e. developer branch)

## Continuous Delivery Example

Next, this KEP explains how the continuous-delivery use case would look for a user.
Therefore, let's assume a user would like to deploy a service `carts` as direct deployment in 
`dev` and as blue/green deployment in `staging` and `production`, i.e.

|       | dev    | staging    | production |
|-------|--------|------------|------------|
| carts | direct | blue/green | blue/green |


**Prerequisites:**
- Install Argo rollouts:
  ```console
  kubectl create namespace argo-rollouts
  kubectl apply -n argo-rollouts -f https://raw.githubusercontent.com/argoproj/argo-rollouts/stable/manifests/install.yaml
  ```
- Install Keptn Argo service (part of CD uniform):
  ```console
  kubectl apply -f https://raw.githubusercontent.com/keptn-contrib/argo-service/0.1.1/deploy/service.yaml
  ```
- Install Keptn Helm-deploy-service (part of CD uniform):
  ```console
  kubectl apply -f https://raw.githubusercontent.com/keptn/helm-deploy-service/master/deploy/service.yaml
  ```

**Preparing the Helm chart:**
- Prepare a Helm chart `carts-direct.tgz` which contains a Deployment manifest for `carts` 
- Prepare a Helm chart `carts-bg.tgz` which contains a Rollout manifest for `carts`. The rollout is similar to a Deployment but allows to specify a `strategy`, i.e.:
  ```console
  apiVersion: argoproj.io/v1alpha1
  kind: Rollout
  metadata:
    name: carts
    labels:
      app: carts
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: carts
    strategy:
      blueGreen:
        autoPromotionEnabled: false
        activeService: carts-primary
        previewService: carts-canary
  ```

**Create a Project:**

The shipyard defines three stages with deployment and release tasks:
```console
apiVersion: spec.keptn.sh/0.2.0
kind: Shipyard
metadata:
  name: sockshop
spec:
  stages:
  - name: "dev"
    workflows:
    - name: artifact-delivery
      triggers:   
      - dev.deployment.triggered
      tasks:
      - name: deployment
      - name: test
          kind: functional
      - name: evaluation
      - name: approval
      - name: release
  - name: "staging"
    workflows:
    - name: artifact-delivery
      triggers:   
      - staging.deployment.triggered
      tasks:
      - name: deployment
      - name: test
          kind: performance
      - name: evaluation
      - name: approval
      - name: release
  - name: "production"
    workflows:
    - name: artifact-delivery
      triggers:   
      - production.deployment.triggered
      tasks:
      - name: deployment
      - name: release
```

Create a Keptn project with
```console
keptn create project sockshop --shipyard=shipyard.yaml 
```

**Creating and onboarding the `carts` service:**

First, create a service carts with
```console
keptn create service carts --project=sockshop
```

Afterwards, onboard the Helm charts with
```console
keptn add-resource  --project=sockshop --stage=dev --service=carts --resource=carts-direct.tgz --resourceUri=helm/carts.tgz
keptn add-resource  --project=sockshop --stage=staging --service=carts --resource=carts-bg.tgz --resourceUri=helm/carts.tgz
keptn add-resource  --project=sockshop --stage=production --service=carts --resource=carts-bg.tgz --resourceUri=helm/carts.tgz
```
Note, adding the Helm chart also for the production namespace can be skipped because we can also use a PR for this.

**Deploy changes of Git repo:**

To trigger a deployment for a certain stage, a CloudEvent of type `sh.keptn.event.deployment.triggered` has to be sent
to the Keptn API.

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

Therefore, the CLI provides the following command:

```console
keptn send event -f event.json
```



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
- Automatically inform Keptn that a branch in e.g. GitHub changed or a PR was merged. But this can also be implemented with Argo CD or Flux.