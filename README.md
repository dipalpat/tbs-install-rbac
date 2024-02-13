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