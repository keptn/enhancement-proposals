# KEP 91: Lifecycle Toolkit - Workload Information in KeptnApp

**State: DRAFTING**

## Motivation
At the moment, the definition of a workload is mainly done in the workload manifests (Deployment, StatefulSets,...) and there is no way to change a workload definition without manipulating this configuration. As the Lifecycle Toolkit also has a definition of the whole Application in a KeptnApp Resource, this leads to the situation that Users have to change two manifests (Deployment and KeptnApp) when they want to use the Keptn Lifecycle Toolkit.

This KEP proposes to add the workload information to the KeptnApp Resource, so that the Lifecycle Toolkit can be configured in a single place.

## Current Design
When configuring a workload, the Lifecycle Toolkit assumes to get information about the Workloads as Annotations/Labels in the Deployment/StatefulSet manifest. This information is used to Link the workload to the corresponding KeptnApp Resource and in further consequence to define which pre- and post-deployment tasks/evaluations should be executed.


## Assumptions / Definitions
* If there are no workload definitions in the Deployment manifests, the Lifecycle Toolkit will wait for a certain timespan to get the workload information from the KeptnApp Resource. If this timespan is exceeded, the LT will assume that the Workload is not managed by Keptn and will not execute any pre- and post-deployment tasks/evaluations.
* If there are workload definitions in the Deployment manifests, the Lifecycle Toolkit will use the information from the Deployment manifests and pre- and post-deployment tasks/evaluations will be merged from the Deployment manifests and the KeptnApp Resource.

## Proposed Design
The KeptnApp Resource will be extended with the following fields:

```yaml
apiVersion: keptn.sh/v1alpha1
kind: KeptnApp
metadata:
  name: my-app
  namespace: keptn
spec:
  version: 1.0.0
  workloads:
  - name: my-workload
    version: 1.0.0
    selector:
      labelSelector:
        matchLabels:
          app: my-workload
          version: 1.0.0
    pre-deployment-tasks: my-pre-deployment-task (optional)
    post-deployment-tasks: my-post-deployment-task (optional)
    pre-deployment-evaluation: my-pre-deployment-evaluation (optional)
    post-deployment-evaluation: my-post-deployment-evaluation (optional)
```

* `selector` defines the selector for the workload. This selector is used to find the corresponding Deployment/StatefulSet manifest.
* `labelSelector` defines that the selection is based on labels.
* `matchLabels` contains a map of labels that are used for the selection. All labels have to match to select the workload.
* `pre-deployment-tasks` defines the name of the pre-deployment task sequence that should be executed before the deployment of the workload.
* `post-deployment-tasks` defines the name of the post-deployment task sequence that should be executed after the deployment of the workload.
* `pre-deployment-evaluation` defines the name of the pre-deployment evaluation that should be executed before the deployment of the workload.
* `post-deployment-evaluation` defines the name of the post-deployment evaluation that should be executed after the deployment of the workload.

When a workload is deployed, the Lifecycle Toolkit will check if there are workload definitions in the Deployment manifests. If there are workload definitions in the Deployment manifests, the Lifecycle Toolkit will use the information from the Deployment manifests and pre- and post-deployment tasks/evaluations will be merged from the Deployment manifests and the KeptnApp Resource. If there are no workload definitions in the Deployment manifests, the Lifecycle Toolkit will use the information from the KeptnApp Resource. If the KeptnApp Resource of a Keptn managed namespace does not appear, the Lifecycle Toolkit will wait for 60 seconds (or any configurable timespan) and if no KeptnApp Resource appears, the LT will assume that the Workload is not managed by Keptn and will not execute any pre- and post-deployment tasks/evaluations.

## Benefits
* No need to define the workload information in two places (Deployment manifest and KeptnApp Resource)
* Also Operator-managed workloads can be managed by Keptn
* Separation of concerns: KeptnApp can be defined by another team as the Workload manifests
