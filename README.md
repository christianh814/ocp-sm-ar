# OpenShift Service Mesh with Argo Rollouts

Example Repo that does what the title says.

# Prereqs

* OpenShift Cluster v4.10+
* [OpenShift Service Mesh (aka Istio) v2.0+](https://docs.openshift.com/container-platform/latest/service_mesh/v2x/preparing-ossm-installation.html)
* [OpenShift GitOps (aka Argo CD) v1.6+](https://docs.openshift.com/container-platform/latest/cicd/gitops/installing-openshift-gitops.html)
* [Argo Rollouts v1.3+](https://argoproj.github.io/argo-rollouts/installation/)


Export Gateway

```shell
export GATEWAY_URL=$(oc -n istio-system get route istio-ingressgateway -o jsonpath='{.spec.host}')
```
Open in browser, example

```shell
firefox $GATEWAY_URL
```
