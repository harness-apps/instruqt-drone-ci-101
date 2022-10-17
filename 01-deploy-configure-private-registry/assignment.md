---
slug: deploy-configure-private-registry
id: wv3kssi1aj6y
type: challenge
title: Deploy and Configure Private Container Registry
teaser: Deploy and Configure a local container registry
notes:
- type: text
  contents: |-
    ## Objectives

    In this challenge, this is what you'll learn:
    - [x] Verify Container Registry Setup
    - [x] Make the private registry available to Kubernetes
    - [x] Run tests to ensure you are able to push and pull from the registry

    ## Duration

    ~10 minutes
tabs:
- title: Terminal
  type: terminal
  hostname: kubernetes-vm
difficulty: basic
timelimit: 600
---

🚀 Introduction
===============

As part of the challenges we will push the  images to container registries. To make it easy for push and pull, we wil use [Container registry](https://docs.docker.com/registry/) from Docker.

🔧 Deploy Local Container Registry
==================================

On the **Terminal** tab navigate to the examples folder,

```shell
cd "$TUTORIAL_HOME"
```

As prompted let us set some environment variables,

```shell
direnv allow .
```

Before we execute the pipeline we need to start a [local container registry](https://docs.docker.com/registry/), run the following command to start one locally,

Run the following command to start a local container registry,

```shell
docker run -d -p 5001:5000 --name registry registry:2
```

You can verify if the registry is started using the `docker ps` command.

```shell
docker ps --filter name=registry
```

The command should return an output like,

<pre>
CONTAINER ID   IMAGE        COMMAND                  CREATED         STATUS         PORTS                                       NAMES
c1d058f61798   registry:2   "/entrypoint.sh /etc…"   5 seconds ago   Up 3 seconds   0.0.0.0:5001->5000/tcp, :::5001->5000/tcp   registry
</pre>

🫙 Configure Private Registry
=============================

The Container registry is visible to the Kubernetes cluster but the container runtime has no clue about, to make the container runtime use it we need to configure that as [k3s private registry](https://rancher.com/docs/k3s/latest/en/installation/private-registry/).

```shell
cat <<EOF > /etc/rancher/k3s/registries.yaml
mirrors:
  "my-registry.localhost:5001":
    endpoint:
      - http://localhost:5001
  "docker.io":
    endpoint:
      - http://localhost:5001
EOF
```

Restart the k3s server to ensure it picks up the registry configuration,

```shell
systemctl restart k3s
```

Wait for kubernetes to be back up and running,

```shell
kubectl get pods -n kube-system
```

The output of the command should be like

<pre>
NAME                                      READY   STATUS      RESTARTS   AGE
local-path-provisioner-7b7dc8d6f5-p8jdb   1/1     Running     0          75m
coredns-b96499967-8vm62                   1/1     Running     0          75m
helm-install-traefik-crd-pk2bw            0/1     Completed   0          75m
helm-install-traefik-4fqwf                0/1     Completed   1          75m
svclb-traefik-89547357-7xclt              2/2     Running     0          75m
metrics-server-668d979685-865cp           1/1     Running     0          75m
traefik-7cd4fcff68-zmcn9                  1/1     Running     0          75m
</pre>

Verify Registry Setup
=====================

Let us try to push an image to the private registry using docker,

```shell
docker pull gcr.io/google-samples/hello-app:1.0
docker tag gcr.io/google-samples/hello-app:1.0 "localhost:5001/hello-app:1.0"
docker push "localhost:5001/hello-app:1.0"
```

Let us use the image that we pushed earlier as part of Kubernetes deployment,

```shell
kubectl create deployment hello-server --image="my-registry.localhost:5001/hello-app:1.0"
kubectl rollout status deployment.apps/hello-server --timeout=30s
kubectl delete deployment.apps/hello-server
```

🏁 Finish
=========

To complete this challenge, press **Check**.
