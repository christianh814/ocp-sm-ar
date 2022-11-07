# OpenShift Service Mesh with Argo Rollouts

Example Repo that does what the title says. You can watch a demo of this repo: [HERE](https://youtu.be/DfeL7cdTx4c)

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

> **NOTE** Errors are expected in this step. Go get some coffee, go for a walk, then comeback to this.

```shell
until oc apply -k bootstrap/overlays/default/; do sleep 15; done
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

Update the `workloads/canary-app/kustomization.yaml` file from `blue` to `yellow`. Edit the file by hand but if you're brave, you can run a `sed` on the file.

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

You'll see that the blue squares turn into yellow ones. It increases every 15 (or so) seconds until you're fully yellow

## Auto-Rollback

In the UI, you'll see a slider that causes the application to return a 500 error

![500err](https://i.ibb.co/LzzWqX4/witherr.jpg)

Change the application back to blue

```shell
sed -i 's/yellow/blue/g' workloads/canary-app/kustomization.yaml
```

Commit and push...

```shell
git add .
git commit -am "blue with errors"
git push
```

Initiate the rollout by refreshing the Argo CD "canary-app" Application. This will initiate a rollout.

Once the rollout has started, slide the error slider on the application to 100% and watch as the rollout fails

```shell
oc argo rollouts get rollout rollouts-demo -n canary
```

It should fail and it should look something like this...

```shell
NAME                                       KIND         STATUS        AGE    INFO
⟳ rollouts-demo                            Rollout      ✖ Degraded    3h27m  
├──# revision:26                                                             
│  ├──⧉ rollouts-demo-6499d5bbb9           ReplicaSet   • ScaledDown  16m    canary,delay:passed
│  └──α rollouts-demo-6499d5bbb9-26        AnalysisRun  ✖ Failed      2m44s  ✖ 3
├──# revision:25                                                             
│  ├──⧉ rollouts-demo-785bb66569           ReplicaSet   ✔ Healthy     41m    stable
│  │  ├──□ rollouts-demo-785bb66569-7gwpj  Pod          ✔ Running     13m    ready:2/2
│  │  ├──□ rollouts-demo-785bb66569-hqmlc  Pod          ✔ Running     13m    ready:2/2
│  │  ├──□ rollouts-demo-785bb66569-rm7j5  Pod          ✔ Running     13m    ready:2/2
│  │  ├──□ rollouts-demo-785bb66569-mjcq5  Pod          ✔ Running     12m    ready:2/2
│  │  ├──□ rollouts-demo-785bb66569-h4tzf  Pod          ✔ Running     12m    ready:2/2
│  │  ├──□ rollouts-demo-785bb66569-v6qn2  Pod          ✔ Running     12m    ready:2/2
│  │  ├──□ rollouts-demo-785bb66569-fvvmb  Pod          ✔ Running     11m    ready:2/2
│  │  ├──□ rollouts-demo-785bb66569-jdt27  Pod          ✔ Running     11m    ready:2/2
│  │  ├──□ rollouts-demo-785bb66569-f6krk  Pod          ✔ Running     11m    ready:2/2
│  │  └──□ rollouts-demo-785bb66569-w7w6w  Pod          ✔ Running     11m    ready:2/2
```

Your app should have failed back to being all yellow since you had errors during the rollout. To restart/fix this. Move the error slider to 0% and restart the rollout.

```shell
oc argo rollouts retry rollout rollouts-demo -n canary
```

This time, the rollout should finish and you should have blue squares.
