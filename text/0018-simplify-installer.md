# Simplify installation - Do not install istio nor nginx-ingress

* Keptn Installer does no longer automatically install istio nor nginx-ingress.
* API and Bridge are no longer exposed using VirtualServices or Ingress objects - instead they are exposed using Kubernetes services (NodePort or LoadBalancer).
* Helm-service (and services that use `KEPTN_DOMAIN`) will become independent from the Keptn API Domain.
* More power to the admin! Exposing Bridge or API is an option for the admin, not for our CLI/API.

## Motivation

With Keptn 0.6.2 the installer has two options: 

- Full Installation (default) - installs istio.
- Control Plane (`--use-case quality-gates`)  - installs nginx-ingress.

Depending on the selected option, Istio or nginx-ingress are installed and various manifests are applied for Keptn API and Bridge. 

Installing 3rd-party software is always risky, and we need to take great care with all our dependencies (whether they are in code or in installing additional software on Kubernetes).

At the moment, we install Istio to demo the full installation of Keptn with onboarded services, blue-green deployments and such, but it is not something that is required to actually run the Keptn control-plane.
In addition, Istio is quite a big software itself, and it's more than normal that security vulnerabilities are discovered (see [istio.io/latest/news/security](https://istio.io/latest/news/security/)). It is impossible to release a new version of Keptn every time a new version of Istio is released. At the same time we tell people to use Keptn, which is risky as users will potentially install an unsecure version of Istio.

We also got some feedback from users regarding this:

* Many users already have istio installed by themselves and asked for an install option to re-use Istio. This was handled via [an experimental flag](https://keptn.sh/docs/0.6.0/reference/managed_istio/). 
* Some users were not allowed to use istio (nor nginx-ingress) and asked on whether we can support alternatives (the answere was unfortunately a no).
* Most Kubernetes users know how to expose a service using Kubernetes services of type LoadBalancer and NodePort - but using Istio was challenging for many. This made troubleshooting much more complicated.
* Many users asked for support of SSL certificates, which was really troublesome in many ways (it's not something Keptn should do, but at the same time we used our self-signed certificates for API and Bridge).

This should be motivation enough to

* get rid of any automated installation of istio and nginx-ingress,
* use Kubernetes service types NodePort/LoadBalancer where possible, and
* let the user handle any service mesh / ingress-gateway related stuff on themselves. 

## Explanation

When installing Keptn, you can choose to expose Keptn API (and, optionally Keptn Bridge via Kubernetes Services.

This can be done via the flags `--keptn-api-service-type=[ClusterIP | NodePort | LoadBalancer]` (and `--keptn-bridge-service-type=[ClusterIP | NodePort | LoadBalancer]`), where `ClusterIP` is the default if nothing is specified.

Depending on the service-type used, Keptn will configure the Kubernetes services of `api-gateway-nginx` and/or `bridge`, which can be verified by executing `kubectl -n keptn get service bridge api-gateway-nginx`.

This would be the default when just using `keptn install` (or `keptn install --keptn-api-service-type=ClusterIP --keptn-bridge-service-type=ClusterIP`):

```
$ kubectl -n keptn get service bridge api-gateway-nginx
NAME                TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
bridge              ClusterIP   10.0.83.209   <none>        8080:32122/TCP   85d
api-gateway-nginx   ClusterIP   10.0.94.251   <none>        80/TCP           41d
```
In this example neither bridge nor API are reachable from the outside. To reach it, one would have to use Kubernetes `port-forward` and re-authenticate as follows:

```console
kubectl -n keptn port-forward svc/api-gateway-nginx 8080:80
```

Now the API is reachable on `http://localhost:8080`, and keptn CLI can be authenticated as follows:

```console
keptn auth --endpoint=http://localhost:8080/ --api-token=$KEPTN_API_TOKEN --scheme=http
```


Here is another example if using `keptn install --keptn-api-service-type=LoadBalancer --keptn-bridge-service-type=ClusterIP`
```
$ kubectl -n keptn get service bridge api-gateway-nginx
NAME                TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)          AGE
bridge              ClusterIP      10.0.83.209   <none>         8080:32122/TCP   85d
api-gateway-nginx   LoadBalancer   10.0.94.251   11.22.33.44    80/TCP           41d
```
In this example, the API is reachable via `http://11.22.33.44:80`, and keptn CLI can be authenticated as follows:

```console
keptn auth --endpoint=http://11.22.33.44:80 --api-token=$KEPTN_API_TOKEN --scheme=http
```


Depending on the selected type, various restrictions might apply, e.g.:

* `LoadBalancer` is not always available and the services external IP stays in status `<Pending>`.
* `NodePort` might expose the service on a high port number (e.g., 31233) that might need to be configured using firewall rules
* `ClusterIP` only exposes the service within the cluster, but not to the outside. This setting is desireable when applying an ingress-manifest manually afterwards. Alternatively, a `port-forward` can be used to reach the service.


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

* CLI/API: Change `keptn configure bridge` no longer provide the flag `action=[expose | lockdown]`, only user and password.
* Docs: Document how to expose Keptn Bridge as well as API using service-type `LoadBalancer` or `NodePort` afterwards (`kubectl patch ...`).

* Upgrader: Research if any of the above changes need to be considered in the upgrader. Note: I think we don't need to change the upgrader for this. The current system should not break if the ugprade is applied, e.g., if bridge is already exposed using nginx-ingress it should stay exposed. If istio-system is already installed, it should stay as it is.

* General: `KEPTN_DOMAIN` is no longer available (the installer can not determine this reliably). We need to research where this might be a problem. For `helm-service` (virtual-services for onboarded services) and `openshift-route-service` (oc create route for onboarded srvices), we could use a separate configmap with a variable called `INGRESS_HOSTNAME_SUFFIX` which is set to `cluster.svc.local` by default.
* Dynatrace-Service: `KEPTN_DOMAIN` is required for configuring problem notifications that are sent back to the Keptn API. We need to come up with a solution for this...

* Docs: Update docs for installation with the preferred service-type for each Cloud Provider 
* Docs: Remove the experimental tutorial about reusing an existing ingress-gateway (e.g., istio)
* Docs/Tutorials: Update docs for installation with a troubleshooting guide if API can not be reached (e.g., refer to Kubernetes services and description of ClusterIP vs. NodePort vs. LoadBalancer) 
* Docs/Tutorials: Provide solid Docs for Demo/POC (e.g., with GKE, K3s) and setting up a full installation of Keptn with onboarding services 

## Trade-offs and mitigations

### Each Kubernetes instance or Cloud-provider has different LoadBalancers (or non at all)

Each platform has different ways of using the service types NodePort and LoadBalancer, but we can tackle this with recommending the preferred option for the respective platform (e.g., GKE, AKS, EKS – Loadbalancer; MiniKube, K3s, MicroK8s: NodePort; OpenShift, MiniShift: ???) 

### Keptn API can not be reached after installation

This is a problem that already occurs sometimes, but in this scenario it could occur much more often. However, in this case we can refer the user to the documentation of Kubernetes services and types, and how to use them to expose a service.

### Change the behaviour afterwards

If the user wants to change this behaviour afterwards, it is as easy as calling `kubectl –n keptn edit service api` (`kubectl –n keptn patch service api …`) and change from ClusterIP to LoadBalancer. 

* Change api service type to LoadBalancer
    ```console
    kubectl patch svc api-gateway-nginx -n keptn -p '{"spec": {"type": "LoadBalancer"}}'
    ```
* Change api service type to NodePort
    ```console
    kubectl patch svc api-gateway-nginx -n keptn -p '{"spec": {"type": "NodePort"}}'
    ```
* Change bridge service type to LoadBalancer
    ```console
    kubectl patch svc bridge -n keptn -p '{"spec": {"type": "LoadBalancer"}}'
    ```
* Change bridge service type to NodePort
    ```console
    kubectl patch svc bridge -n keptn -p '{"spec": {"type": "NodePort"}}'
    ```

If the user wants to use an ingress-controller (e.g., nginx-ingress, Linkerd, istio, traefic) for API/Bridge, the recommended way is to use ClusterIP and apply an ingress manifest manually. 


## Breaking changes

* The CLI command `keptn configure bridge` will need to be simplified to no longer change any service types or ingress manifests.
* The configmap for `KEPTN_DOMAIN` will no longer be available, as we don't know the domain when installing Keptn.
* Helm-Service will break without `KEPTN_DOMAIN` - instead we should ask the user to supply this configuration using a new environment variable called `INGRESS_HOSTNAME_SUFFIX` which we will default to `svc.cluster.local`.
* Helm-Service will need to check if istio is available before applying any virtualservices.
* OpenShift-route-service will need similar changes like the `helm-service` and rely on `INGRESS_HOSTNAME_SUFFIX` when calling `oc create route`.
* It will not be possible to auto-generate deep-links to Keptn Bridge, as we don't have a `KEPTN_DOMAIN` anymore, and Keptn is no longer responsible for exposing Keptn Bridge.
* Dynatrace-service will break as it also relies on `KEPTN_DOMAIN` for problem notifications and hyperlinks to the onboarded services in the Dashboard as well as deep-links to Keptn Bridge.
* Keptn is no longer responsible for generating an SSL certificate (it was self-signed anyway).
* After installation, the user might have to configure the CLI manually in case of problems (e.g., when calling `keptn auth ...`).

## Prior art and alternatives

None

## This is what it looks like to the end-user

Please note: This is not a tutorial, just a sketch of what it looks like to the end-user when installing Keptn after this KEP has been implemented.

Keep in mind that installing a production setup for Istio is not trivial, and the instructions here are only meant for **demo purposes**.

### Full Installation on GKE - Variant 1: Use Google LoadBalancer

1. Ensure that you are connected to the correct Kubernetes cluster using `kubectl get nodes`.
1. Install a recent version of Istio (if not already installed):
   Go to https://istio.io/latest/docs/setup/getting-started/ and follow the guide there.
   Example:
   ```console
   istioctl install --set profile=demo
   ```
1. Install Keptn and automatically expose the API using LoadBalancer (recommended for GKE)
   ```console
   keptn install --keptn-api-service-type=LoadBalancer # --use-case=delivery will most likely be needed for 0.7.x
   ```
1. The installer should be able to automatically detect that the API is available based on the selected service type LoadBalancer, e.g., via http://1.2.3.4/, and configure the Keptn CLI.
   In case this fails (for various reasons), the user would have to do the following:
   ```console
   kubectl -n keptn get svc api-gateway-nginx
   ```
   Note down IP and Port of the `api-gateway-nginx` service (e.g., 1.2.3.4, port 80), and manually authenticate the Keptn CLI, e.g.,
   ```console
   keptn auth --endpoint=http://1.2.3.4:80/ --api-token=$(kubectl get secret keptn-api-token -n keptn -ojsonpath={.data.keptn-api-token} | base64 --decode) --scheme=http
   ```
1. If wanted, expose Keptn Bridge using a LoadBalancer
   ```console
   kubectl patch svc bridge -n keptn -p '{"spec": {"type": "LoadBalancer"}}'
   ```
1. [Determine IP and Port of the istio ingressgateway](https://istio.io/latest/docs/tasks/traffic-management/ingress/ingress-control/#determining-the-ingress-ip-and-ports) using
   ```console
   kubectl -n istio-system get svc istio-ingressgateway
   ```
   Note down IP and Port (e.g., 5.6.7.8, port 80) and set a hostname for it in `INGRESS_HOSTNAME_SUFFIX` in the configmap
   ```console
   kubectl -n keptn create configmap --from-literal=INGRESS_HOSTNAME_SUFFIX=5-6-7-8.nip.io
   ```
1. Follow the tutorial for creating a project and onboarding a service


### Full Installation on GKE - Variant 2: Use Ingress objects

This kind-of rebuilds what keptn 0.6.x will do out of the box.

1. Ensure that you are connected to the correct Kubernetes cluster using `kubectl get nodes`.
1. Install a recent version of Istio (if not already installed):
   Go to https://istio.io/latest/docs/setup/getting-started/ and follow the guide there.
   Example:
   ```console
   istioctl install --set profile=demo
   ```
1. Install Keptn, but do not expose the API:
   ```console
   keptn install --keptn-api-service-type=ClusterIP # --use-case=delivery will most likely be needed for 0.7.x
   ```
   The installer will complain that the API has not been exposed and therefore can not be reached. The user will be shown a link to our docs about how to resolve this matter (e.g., `kubectl port-forward`, changing the service-type afterwards, or using an ingress object).

1. The user now can use [Istio Kubernetes ingress](https://istio.io/latest/docs/tasks/traffic-management/ingress/kubernetes-ingress/) to expose the API. First, they need to [determine IP and Port of the istio ingressgateway](https://istio.io/latest/docs/tasks/traffic-management/ingress/ingress-control/#determining-the-ingress-ip-and-ports) using
   ```console
   kubectl -n istio-system get svc istio-ingressgateway
   ```
   Note down IP and Port (e.g., 5.6.7.8, port 80), and apply it to the following manifest:
   ```yaml
   apiVersion: networking.k8s.io/v1beta1
   kind: Ingress
   metadata:
     annotations:
       kubernetes.io/ingress.class: istio
   name: your-api-keptn-ingress
   namespace: keptn
   spec:
   rules:
   - host: api-keptn.5-6-7-8.nip.io
       http:
       paths:
       - backend:
           serviceName: api-gateway-nginx
           servicePort: 80
   ```
   **Note**: If you want SSL Support, please look into the istio documentation 


1. Manually authenticate the Keptn CLI using the hostname specified above, e.g.,
   ```console
   keptn auth --endpoint=http://api-keptn.5-6-7-8.nip.io/ --api-token=$(kubectl get secret keptn-api-token -n keptn -ojsonpath={.data.keptn-api-token} | base64 --decode) --scheme=http
   ```
1. If wanted, expose Keptn Bridge using the same approach.
   Don't forget to configure authentication for Bridge
   ```console
   keptn configure bridge --user=... --password=...
   ```
1. Finally, re-use the IP and Port of the Istio ingressgateway to set `INGRESS_HOSTNAME_SUFFIX` in the configmap
   ```console
   kubectl -n keptn create configmap --from-literal=INGRESS_HOSTNAME_SUFFIX=5-6-7-8.nip.io
   ```
1. Follow the tutorial for creating a project and onboarding a service


### Control-Plane only Installation on GKE - Variant 1: Use Google LoadBalancer

1. Install Keptn control-plane and automatically expose the API using LoadBalancer (recommended for GKE):
   ```console
   keptn install --keptn-api-service-type=LoadBalancer --use-case=control-plane # Note: the --use-case flag might change in the future
   ```
1. The installer should be able to automatically detect that the API is available based on the selected service type LoadBalancer, e.g., via http://1.2.3.4/, and configure the Keptn CLI.
   In case this fails (for various reasons), the user would have to do the following:
   ```console
   kubectl -n keptn get svc api-gateway-nginx
   ```
   Note down IP and Port of the `api-gateway-nginx` service (e.g., 1.2.3.4, port 80), and manually authenticate the Keptn CLI, e.g.,
   ```console
   keptn auth --endpoint=http://1.2.3.4:80/ --api-token=$(kubectl get secret keptn-api-token -n keptn -ojsonpath={.data.keptn-api-token} | base64 --decode) --scheme=http
   ```
1. If wanted, expose Keptn Bridge using a LoadBalancer
   ```console
   kubectl patch svc bridge -n keptn -p '{"spec": {"type": "LoadBalancer"}}'
   ```
   Don't forget to configure authentication for Bridge
   ```console
   keptn configure bridge --user=... --password=...
   ```
1. Continue with any control-plane related tutorial (e.g., quality-gates, PerfaaS, self-healing, ...)


### Control-Plane only Installation on GKE - Variant 2: Using ingress objects

This is essentially the same as for the full installation, but in this case we do not rely on any specific ingress controller.

AFAIK the following controllers should work:

* istio (>= 1.5)
* nginx-ingress
* traefic (e.g., installed by default on K3s)

1. Install Keptn control-plane, but do not expose the API:
   ```console
   keptn install --keptn-api-service-type=ClusterIP --use-case=control-plane # Note: the --use-case flag might change in the future
   ```
   The installer will complain that the API has not been exposed and therefore can not be reached. The user will be shown a link to our docs about how to resolve this matter (e.g., `kubectl port-forward`, changing the service-type afterwards, or using an ingress object).
1. Depending on your ingress controller, determine IP and Port (or hostname)
1. Create an ingress object for API that looks somehow like this (minor changes might be needed depending on the ingress-controller):
   ```yaml
   apiVersion: networking.k8s.io/v1beta1
   kind: Ingress
   metadata:
     annotations:
       kubernetes.io/ingress.class: INSERT_INGRESS_CLASS_HERE
   name: your-api-keptn-ingress
   namespace: keptn
   spec:
   rules:
   - host: api-keptn.5-6-7-8.nip.io
       http:
       paths:
       - backend:
           serviceName: api-gateway-nginx
           servicePort: 80
   ```
1. Manually authenticate the Keptn CLI using the hostname specified above, e.g.,
   ```console
   keptn auth --endpoint=http://api-keptn.5-6-7-8.nip.io/ --api-token=$(kubectl get secret keptn-api-token -n keptn -ojsonpath={.data.keptn-api-token} | base64 --decode) --scheme=http
   ```
1. If wanted, do the same for exposing Keptn Bridge.
   Don't forget to configure authentication for Bridge
   ```console
   keptn configure bridge --user=... --password=...
   ```
1. Continue with any control-plane related tutorial (e.g., quality-gates, PerfaaS, self-healing, ...)


## Open questions

* Do we want to automatically provide an option to expose Keptn Bridge using `--keptn-bridge-service-type` or should we refer to `keptn configure bridge` as before?
* Or do we actually not want to care about Keptn Bridge at all right now, and not provide any specific service-type parameters?
* Dynatrace-Service: `KEPTN_DOMAIN` is required for configuring problem notifications that are sent back to the Keptn API. We need to come up with a solution for this...
* I am not sure if this approach will work on OpenShift (though it should)


## Future possibilities

* The split of `KEPTN_DOMAIN` and `INGRESS_HOSTNAME_SUFFIX` will allow a separation of control-plane and delivery-plane in the future.
* The fact that api/bridge are no longer relying on a specific ingress gateway will allow easier switching of services meshes/ingress gateways in the future (e.g., linkerd, traefic, nginx-ingress)
* api/bridge could be exposed via the same service endpoint (by re-configuring `api-gateway-nginx`), therefore only requiring a single LoadBalancer
