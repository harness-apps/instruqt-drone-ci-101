---
slug: deploying-gitea
id: cr1o1oyqdq0b
type: challenge
title: Deploy Gitea
teaser: Everything starts with git in CI world
notes:
- type: text
  contents: |-
    This track uses a single node Kubernetes cluster on a sandbox virtual machine.
    Please wait while we boot the VM for you and start Kubernetes.

    ## Objectives

    In this challenge, this is what you'll learn:
    - Deploy Gitea on Kubernetes
    - Expose the deployment with a NodePort service
    - View the exposed `gitea-http` service in your browser
    - Explore the Gitea dashboard

    ## Duration

    ~30 minutes
tabs:
- title: Terminal 1
  type: terminal
  hostname: kubernetes-vm
- title: Gitea
  type: website
  path: /
  url: http://kubernetes-vm.${_SANDBOX_ID}.instruqt.io:30950
  new_window: true
- title: Terminal 2
  type: terminal
  hostname: kubernetes-vm
difficulty: basic
timelimit: 1800
---

👋 Introduction
===============

Our first step in our CI journey is to find a place to host our Git repositories. In this track we will use [Gitea](https://gitea.io), since we can run it locally in the Kubernetes cluster and customize it as needed.

🔧 Install Gitea
================

Add Gitea helm charts

```shell
helm repo add gitea-charts https://dl.gitea.io/charts/
helm repo update
```

Use the helm to deploy Gitea

```shell
envsubst < "$TUTORIAL_HOME/helm_vars/gitea/values.yaml" | helm upgrade \
  --install gitea gitea-charts/gitea \
  --values - \
  --wait
```

👤 Add Gitea User
=================

For all the challenges we will be using the Gitea user `user-01`. The user by default will be configured with the following repositories:

- <https://github.com/harness-apps/drone-ci-101.git>
- <https://github.com/harness-apps/java-fruits-api.git>
- <https://github.com/harness-apps/go-fruits-api.git>
- <https://github.com/harness-apps/fruits-app-ui.git>

Run the following command to create `user-01` and configure the user with the repositories listed above

```shell
kustomize build "$TUTORIAL_HOME/k8s/gitea-config" \
  | envsubst | kubectl apply -f -
```

Wait for the job to complete before proceeding further

```shell
kubectl wait --for=condition=complete --timeout=120s -n drone job/workshop-setup
```

Now you can login to Gitea using Gitea tab using the user `user-01` and password `user-01@123`,

![Gitea Dashboard](../assets/gitea-user-dashboard.png)

🏁 Finish
=========

To complete this challenge, press **Check**.
