# OpenShift Service Mesh with Argo Rollouts

Example Repo that does what the title says.

# Prereqs

* OpenShift Cluster v4.10+
* [Argo Rollout Kubernetes CLI Plugin](https://argoproj.github.io/argo-rollouts/installation/#kubectl-plugin-installation)

This repo assumes that this is a "fresh" cluster. Don't run these steps on a production cluster or a cluster you care about. This is for testing only.

# Testing

Highlevel steps to test. These steps assumes you know Argo CD, Argo Rollouts, Istio, and Kubernetes quite a bit.

## Fork the repo

First, fork this repo. You will need to change the [Application Sets in this directory](components/applicationsets) to point to your fork.

## Deploy the Application

After editing the [Application Sets in this directory](components/applicationsets)  to point to your fork, apply it to your OCP cluster

```shell
until oc apply -k bootstrap/overlays/default/; do sleep 10; done
```

## See the App

You should see the app on your browser; first export your Gateway

```shell
export GATEWAY_URL=$(oc -n istio-system get route istio-ingressgateway -o jsonpath='{.spec.host}')
```

Then open in browser, example

```shell
firefox $GATEWAY_URL
```

You should see this

![sample-app](https://i.ibb.co/G2gY1b5/sample-app.png)

## Make update

Update the `workloads/canary-app/kustomization.yaml` file from `blue` to `yellow`. Edit the file by hand but if you're brave run a sed

```shell
sed -i 's/blue/yellow/g' workloads/canary-app/kustomization.yaml
```

Then commit/push to your fork

```shell
git add .
git commit -am "yellow"
git push
```

## Observe

Get status using the plugin (running a `watch` is helpful)

```shell
oc argo rollouts get rollout rollouts-demo -n canary
```

You'll see that the blue squares turn into yellow ones. It increases every 10 seconds until you're fully yellow
