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
    - [x] Run tests to ensure you are able to push and pull from the registry

    ## Duration

    ~10 minutes
tabs:
- title: Terminal
  type: terminal
  hostname: docker-vm
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

Verify Registry Setup
=====================

Let us try to push an image to the private registry using docker,

```shell
docker pull gcr.io/google-samples/hello-app:1.0
docker tag gcr.io/google-samples/hello-app:1.0 "localhost:5001/hello-app:1.0"
docker push "localhost:5001/hello-app:1.0"
```

🏁 Finish
=========

To complete this challenge, press **Check**.
