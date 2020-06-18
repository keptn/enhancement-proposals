# Simplify installation - Do not install istio nor nginx-ingress

* Keptn Installer does no longer automatically install istio nor nginx-ingress.
* API and Bridge are no longer exposed using VirtualServices or Ingress objects - instead they are exposed using Kubernetes services (NodePort or LoadBalancer).
* Helm-service (and services that use `KEPTN_DOMAIN`) will become independent from the Keptn API Domain.

## Motivation

With Keptn 0.6.2 the installer has two options: 

- Full Installation (default) - installs istio.
- Control Plane (`--use-case quality-gates`)  - installs nginx-ingress.

Depending on the selected option, Istio or nginx-ingress are installed and various manifests are applied for Keptn API and Bridge. 

Installing 3rd-party software is always risky, and we need to take great care with all our dependencies (whether they are in code or in installing additional software on Kubernetes).

At the moment, we install Istio to demo the full installation of Keptn with onboarded services, blue-green deployments and such, but it is not something that is required to actually run the Keptn control-plane.
In addition, Istio is quite a big software itself, and it's more than normal that security vulnerabilities are discovered (see [istio.io/latest/news/security](https://istio.io/latest/news/security/)). It is impossible to release a new version of Keptn every time a new version of Istio is released. At the same time we tell people to use Keptn, which is risky as users will potentially install an unsecure version of Istio.

We also got some feedback from users regarding this:

a) Many users already have istio installed by themselves and asked for an install option to re-use Istio. This was handled via [an experimental flag](https://keptn.sh/docs/0.6.0/reference/managed_istio/). 
b) Some users were not allowed to use istio (nor nginx-ingress) and asked on whether we can support alternatives (the answere was unfortunately a no).
c) Most Kubernetes users know how to expose a service using Kubernetes services of type LoadBalancer and NodePort - but using Istio was challenging for many. This made troubleshooting much more complicated.
d) Many users asked for support of SSL certificates, which was really troublesome in many ways (it's not something Keptn should do, but at the same time we used our self-signed certificates for API and Bridge).

This should be motivation enough to

* get rid of any automated installation of istio and nginx-ingress,
* use Kubernetes service types NodePort/LoadBalancer where possible, and
* let the user handle any service mesh / ingress-gateway related stuff on themselves. 

## Explanation

When installing Keptn, you can choose to expose Keptn API And Keptn Bridge via Kubernetes Services.

This can be done via the flags `--keptn-api-service-type=[ClusterIP | NodePort | LoadBalancer]` and `--keptn-bridge-service-type=[ClusterIP | NodePort | LoadBalancer]` (`ClusterIP` is the default if nothing is specified).

Depending on the service-type used, Keptn will configure the Kubernetes services of `api-gateway-nginx` and/or `bridge`, which can be verified by executing `kubectl -n keptn get service bridge api-gateway-nginx`.

This would be the default when just using `keptn install` (or `keptn install --keptn-api-service-type=ClusterIP --keptn-bridge-service-type=ClusterIP`):

```
$ kubectl -n keptn get service bridge api-gateway-nginx
NAME                TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
bridge              ClusterIP   10.0.83.209   <none>        8080:32122/TCP   85d
api-gateway-nginx   ClusterIP   10.0.94.251   <none>        80/TCP           41d
```
In this example neither bridge nor API are reachable from the outside. To reach it, one would have to 

Here is another example if using `keptn install --keptn-api-service-type=LoadBalancer --keptn-bridge-service-type=ClusterIP`
```
$ kubectl -n keptn get service bridge api-gateway-nginx
NAME                TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)          AGE
bridge              ClusterIP      10.0.83.209   <none>         8080:32122/TCP   85d
api-gateway-nginx   LoadBalancer   10.0.94.251   11.22.33.44    80/TCP           41d
```
In this example, the API is reachable via `http://11.22.33.44:80`.


Depending on the selected type, various restrictions might apply, e.g.:

* `LoadBalancer` is not always available and the services external IP stays in status `<Pending>`.
* `NodePort` might expose the service on a high port number (e.g., 31233) that might need to be configured using firewall rules
* `ClusteriP` only exposes the service within the cluster, but not to the outside. This setting is desireable when applying an ingress-manifest manually afterwards.


## Internal details

* CLI: Remove the install option `--ingress-install-option` and its related code
* CLI: Remove the install option `--gateway` and its related code
* CLI: Introduce new options in the CLI for Keptn Install `keptn-api-service-type` and `keptn-bridge-service-type` which default to `ClusterIP` 
* CLI: After installation, try to access the API via all possible IP addresses that the installer determined. If nothing works, show the user a warning and a link to the troubleshooting guide in our docs.
* CLI: Installation should be considered successful if all deployments/pods have been started. Being able to reach Keptn API is not a strong requirement, and should only print a warning.
* Installer: Remove any manifests and any scripts that make use of Istio or nginx-ingress
* Installer: use a placeholder for the service-type of `api-gateway-nginx` and `bridge`, which is then applied based on the settings `--keptn-api-service-type` and `--keptn-bridge-service-type`
* Installer: Determine all possible IP addresses (and ports) that the user can use to access Keptn API (internal IP, external IP, ...)
* Installer: Do not set `KEPTN_DOMAIN`
* Installer: Do not generate SSL certificates

* CLI/API: Change `keptn configure bridge` no longer provide the flag `action=[expose | lockdown]`, but a flag to configure the service-type [user and password should stay as they are, they are still required]

* Upgrader: Research if any of the above changes need to be considered in the upgrader. Note: I think we don't need to change the upgrader for this. The current system should not break if the ugprade is applied, e.g., if bridge is already exposed using nginx-ingress it should stay exposed. If istio-system is already installed, it should stay as it is.

* General: `KEPTN_DOMAIN` is no longer available (the installer can not determine this reliably). We need to research where this might be a problem. For `helm-service` and `openshift-route-service`, we could use a separate configmap with a variable called `INGRESS_HOSTNAME_SUFFIX` which is set to `cluster.svc.local` by default.
* Dynatrace-Service: `KEPTN_DOMAIN` is required for configuring problem notifications that are sent back to the Keptn API. We need to come up with a solution for this...

* Docs: Update docs for installation with the preferred service-type for each Cloud Provider 
* Docs: Remove the experimental tutorial about reusing an existing ingress-gateway (e.g., istio)
* Docs/Tutorials: Update docs for installation with a troubleshooting guide if API can not be reached (e.g., refer to Kubernetes services and description of ClusterIP vs. NodePort vs. LoadBalancer) 
* Docs/Tutorials: Provide solid Docs for Demo/POC (e.g., with GKE, K3s) and setting up a full installation of Keptn with onboarding services 


From a technical perspective, how do you propose accomplishing the proposal? 

In particular, please explain:

* How would the change impact and interact with existing functionality?
* Likely error modes and how to handle them
* Corner cases and how to handle them

While you do not need to prescribe a particular implementation - a KEP should be about **behavior**, not the implementation - it may be useful to provide at least one suggestion as to how the proposal *could* be implemented. This helps reassure reviewers that implementation is at least possible, and often helps them thinking more deeply about trade-offs, alternatives, etc.

## Trade-offs and mitigations

### Each Kubernetes instance or Cloud-provider has different LoadBalancers (or non at all)

Each platform has different ways of using the service types NodePort and LoadBalancer, but we can tackle this with recommending the preferred option for the respective platform (e.g., GKE, AKS, EKS – Loadbalancer; MiniKube, K3s, MicroK8s: NodePort; OpenShift, MiniShift: ???) 

### Keptn API can not be reached after installation

This is a problem that already occurs sometimes, but in this scenario it could occur much more often. However, in this case we can refer the user to the documentation of Kubernetes services and types, and how to use them to expose a service.

### Change the behaviour afterwards

If the user wants to change this behaviour afterwards, it is as easy as calling `kubectl –n keptn edit service api` (`kubectl –n keptn patch service api …`) and change from ClusterIP to LoadBalancer. 

If the user wants to use an ingress-controller (e.g., nginx-ingress, Linkerd, istio, traefic) for API/Bridge, the recommended way is to use ClusterIP and apply an ingress manifest manually. 




## Breaking changes


* The CLI command `keptn configure bridge` will need to be re-thought.
* The configmap for `KEPTN_DOMAIN` will no longer be available, as we don't know the domain when installing Keptn.
* Helm-Service will break without `KEPTN_DOMAIN` - instead we should ask the user to supply this configuration using a new environment variable called `INGRESS_HOSTNAME_SUFFIX` which we will default to `svc.cluster.local`
* Helm-Service will need to check if istio is available before applying any virtualservices
* Not sure, but OpenShift support might break and we need to fix certain things
* Dynatrace-service will break as it also relies on `KEPTN_DOMAIN` - especially for problem notifications
* Keptn is no longer responsible for generating an SSL certificate
* The user might have to configure the CLI manually

## Prior art and alternatives

None

## Open questions

* Do we want to automatically provide an option to expose Keptn Bridge using `--keptn-bridge-service-type` or should we refer to `keptn configure bridge` as before?
* Dynatrace-Service: `KEPTN_DOMAIN` is required for configuring problem notifications that are sent back to the Keptn API. We need to come up with a solution for this...
* I am not sure if this approach will work on OpenShift
* The installer could be able to determine the IP address that the user is going to be able to access the API, but it might not be reliable...


## Future possibilities

* The split of `KEPTN_DOMAIN` and `INGRESS_HOSTNAME_SUFFIX` will allow a separation of control-plane and delivery-plane in the future.
* The fact that api/bridge are no longer relying on a specific ingress gateway will allow easier switching of services meshes/ingress gateways in the future (e.g., linkerd, traefic, nginx-ingress)
* api/bridge could be exposed via the same service endpoint (by re-configuring `api-gateway-nginx`), therefore only requiring a single LoadBalancer
