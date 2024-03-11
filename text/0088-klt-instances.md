# KEP 88: Adding Instances to KeptnApp 

**State: DRAFTING**

## Motivation
At the moment, the lifecycle toolkit and its resources are operating in a single instance mode, meaning that the lifecycle toolkit is only able to manage a single instance of a workload and no distinction is made between multiple instances (stages/environments) of the same workload. 

This KEP describes the changes required to enable the lifecycle toolkit to manage multiple instances of a workload / application.

## Current Design
Currently, the Keptn Lifecycle Toolkit (Controller) is managing a single instance of an application or workload under the assumption that it is running in a single namespace and if multiple instances exist, there is some distinction between them also in the observability solution.

Therefore, there are no explicit references to the instance of the application in the lifecycle toolkit resources.


## Assumptions / Definitions
- In the Keptn Lifecycle Toolkit, an instance describes a single instance of a workload or application. This can be a single stage or environment of a workload or application.
- An instance has to be specified on the workload and application level
- If an instance is not specified, the lifecycle toolkit will assume that there is only a single instance of the workload or application
- If an instance is not specified on the workload level, the lifecycle toolkit will lookup the corresponding application and use the instance specified there
- Properties of an instance are defined in the `KeptnApp` resource
- Promotion information can be specified in the Instance, there can be successors, but no predecessors of an instance
- This KEP does not cover the promotion of an instance, this will be covered in a separate KEP / Issue 

## Proposed Design
To add the ability to manage multiple instances of an application, the following structure will be added to the `KeptnApp` resource:

```yaml
apiVersion: keptn.sh/v1alpha3
kind: KeptnApp
metadata:
  name: podtatohead
  namespace: keptn
spec:
  instance: <name of the current instance>
  [...]
  instances:
    defaults:
      repository: <git repository>
      branch: <git branch>
      path: <path to the manifest>
      reposecret: <secret to access the repository>
    instances:
    - name: dev
      successor: 
      - staging
      ( repository: <git repository> )
      ( branch: <git branch> )
      ( path: <path to the manifest> )
      ( reposecret: <secret to access the repository> )
    - name: staging
      successor: 
      - production
    - name: production

```

The `instance` property is used to specify the current instance of the application. If this property is not specified, the lifecycle toolkit will assume that there is only a single instance of the application called "single".

The `defaults` section is used to specify the default values for the properties of an instance. If an instance does not have an override for a property, the value from the `defaults` section is used. In the defaults section, the following properties can be specified:
* repository: The git repository where the deployment configuration is located
* branch: The branch of the git repository where the deployment configuration is located
* path: The path to the deployment configuration
* reposecret: The secret to access the git repository

The `instances` map is used to specify the instances of the application. The `instances` property inside is a list of instances. Each instance has a `name` property which is used to identify the instance. The `name` property is used to identify the instance in the `KeptnAppVersion` resource. There can be a list of successors. These successors are used to specify the promotion of an instance. Every instance might have overrides for the properties defined in the `defaults` section which are optional in the instance definition.

The information defined here is also passed over to the KeptnAppVersion resource. Therefore, the KeptnAppVersion resource will be extended with exactly the same properties.

To match the KeptnAppVersion with the respective workloads, following annotations/labels will be supported in the Workload resource:

```yaml
    keptn.sh/instance: <name of the KeptnApp resource>
    app.kubernetes.io/instance: <name of the KeptnApp resource>
```

The KeptnWorkload and KeptnWorkloadVersion resources will be extended with the following properties:

```yaml
apiVersion: keptn.sh/v1alpha3
kind: KeptnWorkload
metadata:
  name: podtatohead
  namespace: keptn
spec:
  instance: <instance name>
  [...]
```

As a result, the determination of the status of a KeptnAppVersion resource as well as a KeptnWorkloadInstance will be extended with the instance name. If there is no instance specified, the lifecycle toolkit will assume that there is only a single instance of the workload called "single".

Finally, the instance information should be added to the spans and metrics of the lifecycle toolkit.

## Benefits
- The lifecycle toolkit will be able to manage multiple instances of a workload or application
- Multiple Instances of a workload or application can be distinguished in the observability solution
- There is enough metadata to determine the promotion of an instance





