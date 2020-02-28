# First version of a Keptn uniform

The Keptn needs a uniform, the definition of HOW it has to work.

## Motivation

*Separation of concerns* is a key principle Keptn follows. Thus, Keptn separates the process defined by Site Reliability Engineers (SREs) from the actual tooling defined by DevOps engineers. These definitions can be managed independently. While an SRE already can use a Shipyard to define processes, a DevOps engineer has no means to declare the tooling. To address this problem, this KEP proposes the first version of the *Uniform*, which declares a list of Keptn-services* that act on events.

*) Remark: A Keptn-service is the executable unit of a task. It is triggered by an event of a task and it has to notify about its *start* and *finished* status using events. 

Please read this KEP in combination with the KEP: [The next generation of Shipyard]().

## Explanation

:man:/:blonde_woman: The target persona of this KEP is a *DevOps Engineer*, who is responsible for the tooling applied to automate application delivery or remediation operations.  

*Current situation and Problem(s):* When installing Keptn, the installation defines the initial toolset a DevOps engineer can use to assemble the delivery or auto-remedation workflows. However, the following problems occur:  

- Most likely, this toolset is not complete or other tools are used for certain tasks, e.g., tests are not implemented in JMeter, but rather in Selenium. To exchange one tool by another, the DevOps engineer has to manually delete the unused Keptn-service with `kubectl delete` and has to deploy the other Keptn-serivce with `kubectl apply`. Not enough, in Keptn 0.6.0 the user has to take care of updating an event distributor to forward an event to the new Keptn-service.

- A DevOps engineer gets a hard time keeping track of toolset changes and verifying how the default Keptn installation has changed over time. 

*Solution:* The **uniform** is the means to describe the required tools a DevOps engineer wants to have available. 

## Internal details

### Specification

The KEP proposes an initial specification of a uniform as explained below. To start the explanation, the following example of a uniform with two *Keptn services* is provided: 

```yaml
---
version: 0.1.0
kind: Uniform
metadata:
  name: uniform-abc
spec:
  services:
    - name: deployment-service
      image: keptn/helm-service:0.6.0
      events:
        - deployment.triggered
      env:
        - name: ENVIRONMENT
          value: 'production'

    - name: test-service
      image: keptn/selenium-service:0.6.0
      events:
        - test.triggered
```

*Meta-data:*
* **version**: The version of the uniform specification. 
* **kind**: Is set to `Uniform`.
* **metadata:** Contains at least the property *name*, which declares a unique name for the uniform.
* **spec:** Consists of the property *services*.

*Defintion of Services:*
* **services:** An array of *Keptn-services*. Each *Keptn-services* consists of the properties name and image, as well as of the arrays events and env. 
* **name**: A unique name of the *Keptn-service*. 
* **image**: A container image that represents the implementation of this *Keptn-service*. 
* **events:** An array of events the *Keptn-service* can process.
* **env:** An array of environment variables used to configure the *Keptn-service*. 

### Scope

A uniform is defined on the level of a project meaning that each project has its uniform.

To apply a uniform, the following command needs to be provided:

```console
keptn wear uniform uniform.yaml --project=xyz
``` 

### Event selector

For this part, please first read the KEP: [The next generation of Shipyard](); especially regarding the eventing part.

An implementation of a Keptn-service is designed for a particular delivery or remedation puporse and not for "I can do everything". For example, a deployment service can handle a direct (aka. recreate) deployment but no blue/green deployment; or a testing service can execute UI tests but no functional API tests or performance tests. 

To provide the mechanism to restrict the events a Keptn-service acts upon, this KEP proposes the concept of an event selector as shown by the following example: 

```yaml
---
version: 0.1.0
kind: Uniform
metadata:
  name: uniform-abc
spec:
  services:
    - name: deployment-service
      image: keptn/helm-service:0.6.0
      events:
        - deployment.triggered:
            selector:
              matchLabels:
                strategy: direct
```

If this selector is configured, the Keptn-service (deployment-service) has a subscription to a `deployment.triggered` event, which has to CONTAIN the label: `strategy: direct`. Consequently, the Keptn-service will listen to just this labeled event to make sure that it performs the action it is built for.

Assuming the above *deployment-service* example can handle multiple deployment strategies, e.g. direct and blue/green, meaning that it has to select events based on the label: `strategy: direct` AND `strategy: blue_green`. For this situation, the above approach does not work since the key `strategy` can not be used for another value. In other words, the below example is not allowed:

```yaml
...
events:
  - deployment.triggered:
      selector:
        matchLabels:
          strategy: direct
          strategy: blue_green # The key `strategy` is already used. This does not work.
```

As a result, this KEP proposes a second approach of an event selector using `matchExpressions`. `MatchExpressions` allows defining a query to select events based on the same label key but different values. This approach is taken from K8s see [here](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#resources-that-support-set-based-requirements). 

```yaml
...
events:
  - deployment.triggered:
      selector:
        matchExpressions:
          - {key: strategy, operator: In, values: [direct, blue_green]}
```

### Impact on existing functionality

- The *Keptn-services* that are responsible for a delivery or remediation workflow (these services are known as batteries), need to be removed from the installation process of Keptn. Instead, these services must move to a default uniform the user can adapt and apply.

## Trade-offs and mitigations

N/A

## Breaking changes

No breaking changes - This KEP proposes addresses the separation of concerns and provides the flexibility to exchange tools. When the default uniform (containing the Keptn-services from Keptn 0.6.0) is applied, there is no difference to a default Keptn 0.6.0 installation.

## Prior art and alternatives

N/A

## Open questions

- How to handle the situation when two projects declare the same *Keptn-service*, but with different listening events or environment variables? 

## Future possibilities

- A future change this proposal enables is the separation of Keptn's control plane from Keptn`s execution/delivery plane. In other words, the Keptn core (Keptn's control plane) can run on Kubernetes cluster A, while the supporting Keptn-services in the uniform (Keptn's delivery plane) can run on Kubernetes cluster B. Cluster B could be the cluster where the actual deployment/testing/releasing is happening.
