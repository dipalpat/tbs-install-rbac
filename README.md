# TBS Install with non-admin user
The RBAC documented in this repo is refinement of RBAC documented on TBS Documentation site. This RBAC and Overlay combined removes the need for Cluster Admin privileges needed to install TBS.

This branch removed  `ca-cert-injection` and `stacks-operator` resources and related RBAC. Additionally, this does not use `leases` resource and removed `patch` and `update` permissions from `serviceaccounts`. 

Adjust the Rolebindings and ClusterRoleBinding to the subject you are using in your setup accordingly. This repo assumes you are using default namespace and default service account to execute the installation. 

Apply the RBAC using Admin Privileges

```
kubectl apply -f tbs-install-rbac.yaml
```
Create Service Account Token
```
kubectl config set-credentials default --token $(kubectl create token default)
```
Use the Service Account Token with existing kubeconfig
```
kubectl config set-context --current --user default
```
Execute the install by Service Account User with limited RBAC
```
ytt -f /tmp/bundle/config/ \ 
  -f ./tbs-overlay.yaml \
  -v kp_default_repository='REPO' \
  -v kp_default_repository_username='REPO_USERNAME' \
  -v kp_default_repository_password='REPO_PASSWORD' \
  --data-value-yaml pull_from_kp_default_repo=true \
  | kbld -f /tmp/bundle/.imgpkg/images.yml -f- \
  | kapp deploy -a tanzu-build-service -f- -y
```
To delete use the following command
```
kapp delete -a tanzu-build-service
```

# Resources and its usage

| Type | Scope | Name | Usage |
| ---- | ---- | ----- | ----- |
| User Role | ClusterRole | build-service-admin-role | admin role for managing kpack.io resources i.e. clusterbuilders, clusterstacks and clusterstores | 
| User Role | ClusterRole | build-service-app-operator-cluster-access | app operator role for managing kpack.io resources i.e. clusterbuilders, clusterbuildpacks, clusterstacks and clusterstores | 
| User Role | ClusterRole | build-service-user-role | this should not be used | 
| User Role | ClusterRole | build-service-cluster-resource-user-role | app dev/user role to get, list, watch cluster scoped builders, stacks and stores | 
| User Role | ClusterRole | build-service-namespaced-resource-user-role |  app dev/user role to manage image, build, namespaced scoped builders |
| User Role | ClusterRole | build-service-app-editor | allows a user to retrigger builds | 
| User Role | ClusterRole | build-service-app-viewer | allows a user to vew namespaced kpack resources (builds, images, builders, buildpacks) | 
| User Role | ClusterRole | build-service-app-operator | app operator role for managing namespaced kpack.io resources | 
| User Role | ClusterRole | build-service-app-viewer-cluster-access | allows a user to view cluster kpack resources i.e. clusterbuilders, clusterbuildpacks, clusterstacks and clusterstores  | 
| User Role | ClusterRole | build-service-namespaced-resource-user-base-role | this is a subrole that is used to aggreagate permissions | 
| User Role | Role | kpack/build-service-admin-configmap-role |  used to give admin users access to modify kpack internal configmaps | 
| User Role | Role | kpack/build-service-authenticated-kp-config-viewer | this role is deprecated | 
| Control Plane Role | ClusterRole | build-service-dependency-updater-role | used for dependency updater manage kpack resources as well as dependency updater resources | 
| Control Plane Role | Role | kpack/build-service-dependency-updater-kp-config-role | allows the dependency updater to read the kp config configmap | 
| Control Plane Role | ClusterRole | kpack-controller-admin | allows the kpack controller to manage all of its crds as well as create and delete pvc and pods for builds | 
| Control Plane Role | ClusterRole | kpack-controller-servicebindings-cluster-role | gives the kpack controller access to service binding provisioned service types | 
| Control Plane Role | Role | kpack/kpack-controller-local-config | allows the kpack controller to read the configmaps in its namespace containing its configuration |
| Control Plane Role | Role | kpack/kpack-webhook-certs-admin | allows the webhook to manage its certificate secret to communicate with the api server | 
| Control Plane Role | ClusterRole | kpack-webhook-mutatingwebhookconfiguration-admin | allows the kpack webhook to inject its roles into the mutatingwebhookconfiguration | 
| Control Plane Role | ClusterRole | build-service-secret-syncer-role | permissions used for the secret syncer, can be disabled if not using | 
| Control Plane Role | ClusterRole | build-service-warmer-role | allows the build service smartwarmer to watch builders | 
| Control Plane Role | Role | build-service/build-service-warmer-namespace-role | allows the build service smartwarmer to create daemonsets | 

