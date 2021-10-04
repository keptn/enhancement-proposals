# Authorization for Keptn-managed secrets using Kubernetes RBAC mechanisms

## Status quo
Keptn services like the webhook-service or services that integrate with 3rd parties need access to Kubernetes secrets.
For those secrets, we do not know the names as they are added by the user.
Hence, those services currently have a Kubernetes role which gives them the permission 
to read any secret within the namespace where Keptn is installed.

## Problem
This role allowing to read any secret poses a security risk.
For example, the webhook-service could be used to read any secret in
the Keptn namespace and send the content to any server.

## Proposed solution
The overall idea is to use Kubernetes RBAC mechanisms to protect the access of secrets:
- The secret-service manages Roles and RoleBindings when creating/editing/deleting a secret.
- The services consuming the secrets require a ServiceAccount, which allows reading the secret.

More precisely, when a new secret is created/edited/deleted, the secret-service also has to create 
(1) a role that allows reading this secret (implementation already exists) and 
(2) a role binding which maps the role to a service account (the “scope”).
Then, the service consuming the secret (e.g. the webhook-service) reads
the secret via the Kubernets API where it is checked whether the associated service account
has the required role.

## Open questions/Implementation details
- Was it planned to map the "scope" provided in the secret endpoints
to Kubernetes Service Accounts when K8s secrets are used for storing the secrets? Or is the scope something different?
--> We identified that we can map the scope to a K8s service account.
- Scopes are currently statically configured using a `scopes.yaml` file and 
   this file only contains the "keptn-default" scope. How can we support arbitrary scopes? 
--> For the first version, we stick to statically configured scopes in the scopes.yaml file. Additionally, we will add a scope for the webhook-service and for the dynatrace-service.
