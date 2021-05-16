# First version of a Keptn Uniform

The Keptn needs a Uniform - the definition of HOW a Keptn executes continuous delivery or automated operations tasks.

## Motivation

*Separation of concerns* is a key principle Keptn follows. Thus, Keptn separates the process defined by Site Reliability Engineers (SREs) from the actual tooling defined by DevOps engineers. These definitions can be managed independently. While an SRE already can use a Shipyard to define processes, a DevOps engineer has no means to declare the tooling. To fill this gap, this KEP proposes the first version of the *Uniform*, which declares a list of Keptn-services* that act on certain events and execute the task they are implemented for.

*) **Definition:** A *Keptn-service* is the executable unit of a task. It is triggered by an event and it has to notify about its *start* and *finished* status using events. 

Please read this KEP in combination with the KEP: [The next generation of Shipyard](https://github.com/keptn/enhancement-proposals/pull/6).

## Explanation

:man:/:blonde_woman: The target persona of this KEP is a **DevOps Engineer**, who is responsible for the tooling to automate continuous delivery or remediation operations.  

*Current situation and Problem(s):* When installing Keptn, the installation defines the initial toolset a DevOps engineer can use to assemble the delivery or operations workflows (aka. task sequences). 

However, the following situations occur:  

- Most likely, this toolset is not complete or other tools are used for certain tasks, e.g., tests are not implemented in JMeter, but rather in Selenium. To exchange one tool by another, the DevOps engineer has to manually delete the unused Keptn-service with `kubectl delete` and has to deploy the other Keptn-service with `kubectl apply`. Not enough, in Keptn 0.6.0 the user has to take care of updating an event distributor to forward an event to the new Keptn-service.

- A DevOps engineer gets a hard time keeping track of toolset changes and verifying how the default Keptn installation has changed over time. 

*Solution:* The **Uniform** is the means to describe the required tools a DevOps engineer wants to have available. 

- A Keptn user needs the possibility to specify the used tool set at least on a Keptn project-level. Now the used tool set is the same for a Keptn installation. For now let us define the uniform on a project-level and not on a stage or service level, because a user can always use the task name in the shipyard to control which tool is used, e.g. test-performance could trigger JMeter tests and test-functional could trigger Selenium tests.

## Service requirements

**Motivation:** The control plane should not be responsible for managing (e.g. starting, stopping, registering) the execution plane services because it most likely does not have the rights to start a service in, e.g., a production environment. Furthermore, it should be transparent for the control plane which execution plane services are used. The control plane only needs to know how many execution plane services are listening for a task in order to do the synchronization.

**Technical approach:**

- A Keptn-service has to register at the control plane. This is required to inform the control plane about its existence and the topic and project the service is interested in. This can be done using a POST request to the shipyard controller (POST KEPTN_ENDPOINT/v1/events/register)

- Each execution plane service must periodically (e.g. 30 sec interval) confirm its interest in the registered topic. 

- Q: When following the CRD approach (see below) should this information be persistent in Git? (To have an audit, when/which service was available) 

## Possible approaches

To implement the concept of a Uniform, two approaches fit into the principles of Keptn: 

- Uniform as Custom Resource Definition (CRD)
- Uniform based on GitOps approach

### Uniform as Custom Resource Definition (CRD)

This approach has actually two expansion stages: 

- Install a Uniform by the plain deployment manifests of the Keptn-services - configured for topic, Keptn endpoint, and project
- Define a CRD that adds “syntactic sugar” to make the handling of the deployment manifest of a Keptn-service easier - focus on the declaration of: image, topic, Keptn endpoint, and project 
  - A CRD concludes to have an Operator that requires RBAC rules to create/delete deployment and service
 
*Known Impacts:*

- The uniform operator can only run on K8s. How do we handle execution plane services not running on K8s?
- The uniform is not Git-managed
 
#### Keptn-service Deployment manifest

As a user, I apply a deployment manifest using: kubectl apply -f xyz.yaml for each Keptn-service.
 
A deployment manifest for a Keptn-service looks as follows:
 
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jmeter-service
  namespace: keptn
  labels:
    app.kubernetes.io/name: jmeter-service
    app.kubernetes.io/instance: keptn
    app.kubernetes.io/part-of: keptn-keptn
    app.kubernetes.io/component: execution-plane
    app.kubernetes.io/version: develop
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: jmeter-service
      app.kubernetes.io/instance: keptn
  replicas: 1
  template:
    metadata:
      labels:
        app.kubernetes.io/name: jmeter-service
        app.kubernetes.io/instance: keptn
        app.kubernetes.io/part-of: keptn-keptn
        app.kubernetes.io/component: execution-plane
        app.kubernetes.io/version: develop
    spec:
      containers:
      - name: jmeter-service
        image: keptn/jmeter-service:0.7.2         # <- image
        livenessProbe:
          httpGet:
            path: /health
            port: 10999
          initialDelaySeconds: 5
          periodSeconds: 5
        ports:
        - containerPort: 8080
      - name: distributor
        image: keptn/distributor:0.7.2
        livenessProbe:
          httpGet:
            path: /health
            port: 10999
          initialDelaySeconds: 5
          periodSeconds: 5
        ports:
          - containerPort: 8080
        resources:
          requests:
            memory: "32Mi"
            cpu: "50m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        env:
          - name: PUBSUB_URL
            value: 'nats://keptn-nats-cluster'     # <- *not needed anymore*
          - name: PUBSUB_TOPIC
            value: 'sh.keptn.event.test.triggered' # <- topics
          - name: PUBSUB_RECIPIENT
            value: '127.0.0.1'
          - name: KEPTN_ENDPOINT                   # <- *new* endpoint
            value: 
          - name: PROJECTS                         # <- *new* project
            value: []
```
 
A (simple) user experience improvement would be to provide an Umbrella Helm Chart for frequently used services in the execution plane, e.g. a simple version of a market place. This would allow the Keptn users to install/deinstall Keptn services by setting SERVICE-NAME.enabled=true/false in the values file of the Umbrella chart.
This requires to provide a Hem chart for each Keptn service.

#### Custom Resource Definition
 
As a user, I apply the CRD using: kubectl apply -f uniform.yaml to deploy the entire execution plane.
 
A specification of a Custom Resource Definition (CRD) for a Uniform looks as shown below. To get started, the following example of a uniform with two *Keptn-services* is provided: 

```yaml
---
version: 0.1.0
kind: Uniform
metadata:
  name: uniform-abc
spec:
  endpoint:                            # <- endpoint 
  project:                             # <- project 
  services:
    - name: deployment-service
      image: keptn/helm-service:0.6.0  # <- image
      events:                          # <- topics
        - deployment.triggered
      env:
    - name: test-service
      image: keptn/selenium-service:0.6.0
      events:
        - test.triggered
```

*Meta-data:*
* **version**: The version of the uniform specification. 
* **kind**: Is set to `Uniform`.
* **metadata:** Contains at least the property *name*, which declares a unique name for the uniform.
* **spec:** Consists of the property `services`.

*Definition of Keptn-Services:*
* **services:** An array of *Keptn-services*. Each *Keptn-services* consists of the *name* and *image* property, as well as of the *events* and *env* array:
  * **name**: A unique name of the *Keptn-service*. 
  * **image**: A container image that represents the implementation of this *Keptn-service*. 
  * **events:** An array of events the *Keptn-service* can process.
  * **env:** An array of environment variables used to configure the *Keptn-service*.
 
### Uniform based on GitOps-approach

As a user, I add a `uniform.yaml` (as shown by CRD) to the Git repository of the project.
On the execution plane, an “operator” detects this change and automatically applies it to the target platform. 

*Known Impacts:*

- Either the uniform “operator” needs access to the Git repository or we will provide an endpoint at the control plan, which allows the uniform operator to query the uniforms.
- The uniform operator needs to know for which project it is responsible for.

*Format:*

- We can reuse the same format as we do for the CRD-approach. 

*Unsorted thoughts:*

- When we pick this approach, the Git repository should be the single place where to change the tooling, e.g. no changes directly in the uniform “operator”

## Impact on existing functionality

- Keptn 0.7.0 installation process: The *Keptn-services* that are responsible for a delivery or remediation workflow (aka, batteries such as: jmeter-service, lighthouse-service, remediation-service, etc.), need to be removed from the installation process of Keptn. Instead, these services must move to a default uniform the user can adapt and apply.

## Trade-offs and mitigations

N/A

## Breaking changes

No breaking changes - This KEP addresses the separation of concerns and provides the flexibility to exchange tools. When the default uniform (containing the Keptn-services from Keptn 0.7.0) is applied, there is no difference to a default Keptn 0.7.0 installation.

## Prior art and alternatives

N/A

## Open questions

- Uniform on project level: **What are the implications from an architectural perspective?** > How to handle the situation when two projects declare the same *Keptn-service*, but with different listening events, versions, or environment variables?

- Q: How to deal with "provider" services, e.g., sli-provider or remediation-provider. How to configure these services in the uniform? From an architectural perspective, are calls to these services synchronously or asynchronously? 
  - A: SLI providers will become usual Keptn services with no special treatment.
