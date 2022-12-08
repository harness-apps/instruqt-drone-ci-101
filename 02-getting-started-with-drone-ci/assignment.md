---
slug: getting-started-with-drone-ci
id: zaiaj9p9de4v
type: challenge
title: Getting started with Drone CI
teaser: Introduction to Drone CI
notes:
- type: text
  contents: |-
    ## Objectives

    In this challenge, this is what you'll learn:
    - [x] Understand the structure of `.drone.yml`
    - [x] Create your first Drone pipeline
    - [x] Run your Drone pipeline using `drone` CLI
    - [x] Build and push a container image to local container registry
    - [x] Using environment variables with pipelines
    - [x] Using secrets with pipelines
    - [x] Understanding Plugins and Services
    - [x] Mounting Volumes from the Docker host

    ## Duration

    ~30 minutes
tabs:
- title: Terminal
  type: terminal
  hostname: kubernetes-vm
- title: Editor
  type: code
  hostname: kubernetes-vm
  path: /root/repos/drone-ci-101
- title: Terminal 2
  type: terminal
  hostname: kubernetes-vm
difficulty: basic
timelimit: 1800
---

👋 Introduction
===============

Drone is very lightweight CI that can run in both Client-Server mode or as arbitrary binary on our local laptops.

Download and Install Drone CLI
==============================

Let us download and install Drone CLI,

```shell
curl -L https://github.com/harness/drone-cli/releases/download/v1.6.2/drone_linux_amd64.tar.gz| tar zx
install -t /usr/local/bin drone
```

Check if you are able to access and use `drone` binary,

```shell
drone --version
```

The command should show an output like `drone version 1.6.2`.

Ensure Environment
==================

```shell
docker --version
```

The command should show an output like `Docker version 20.10.21, build baeda1f`

Writing Pipelines
==================


Your first Pipeline
-------------------

In order get the feel of `drone` let us run create and run our very first pipeline.

On the **Terminal** tab navigate to the tutorial home folder,

```shell
cd "$TUTORIAL_HOME"
```

As prompted let us set some environment variables,

```shell
direnv allow .
```

Navigate to `Editor` tab and create a new folder called `my-first-pipeline`.

In the `my-first-pipeline` create a file `.drone.yml` with the following content,

```yaml
kind: pipeline
type: docker
name: default

steps:
- name: say hello
  image: busybox
  commands:
  - echo 'Hello World'
```

Let us run our first pipeline, navigate to the **Terminal** tab and run,

```shell
cd $TUTORIAL_HOME/my-first-pipeline
drone exec
```

The command should run our pipeline and shown an output like,

```text
[say hello:0] + echo 'Hello World'
[say hello:1] Hello World
```

**Congratulations!**. You have successfully run your first pipeline.

Pipeline **steps** are run sequentially. Update the `.drone.yml` on the **Editor** tab as shown below.

```yaml
kind: pipeline
type: docker
name: default

steps:
- name: say hello
  image: busybox
  commands:
  - echo 'Hello World'
- name: good bye
  image: busybox
  commands:
  - echo 'Good bye'
```

Try running the `drone` command again,

```shell
drone exec
```

The pipeline run shows the following output,

```text
[say hello:0] + echo 'Hello World'
[say hello:1] Hello World
[good bye:0] + echo 'Good bye'
[good bye:1] Good bye
```

Trusted mode
------------

Trusted mode instructs `drone` to run in pipelines in privileged mode.

When to use **trusted** mode ?

Lets take an example of building and pushing our application as a container image to local registry. In order to do that we need,

- An application, with Dockerfile to build the container image
- A local container registry, where the built image will be pushed. We already have container registry that we deployed as part of the first challenge.
- Mount local folders onto the pipeline containers

Pipeline
--------

On the **Terminal** tab navigate back to `$TUTORIAL_HOME/go-hello-world` folder

```shell
cd $TUTORIAL_HOME/go-hello-world
```

Open the exercise folder **$TUTORIAL_HOME/go-hello-world** on the **Editor** and analyse the Drone pipeline,

```yaml
kind: pipeline
type: docker
name: default

steps:

- name: build-image
  image: plugins/docker
  settings:
    repo: localhost:5001/example/go-hello-world
    insecure: true
  environment:
    # enable docker buildkit
    # https://docs.docker.com/develop/develop-images/build_enhancements/#to-enable-buildkit-builds
    DOCKER_BUILDKIT: "1"
  volumes:
    - name: docker-sock
      path: /var/run/docker.sock

volumes:
- name: docker-sock
  host:
   path: /var/run/docker.sock
```

It is a simple one step pipeline that builds the go application using the *Dockerfile*:

```dockerfile
#syntax=docker/dockerfile:1.3-labs

FROM golang:1.18-alpine
ARG TARGETARCH

ENV CGO_ENABLED=0

RUN apk add --update

WORKDIR /build

COPY . .

RUN --mount=type=cache,target=/go/pkg/mod \
    --mount=type=cache,target=/root/.cache/go-build \
    GOOS=linux GOARCH=${TARGETARCH} go build -o server server.go

CMD [ "/build/server" ]
```

And pushes the image to local container registry `localhost:5001` as *localhost:5001/example/go-hello-world:latest*.

> **NOTE**:
> The docker build is done using Drone **Plugins**, we will talk more about it in upcoming chapters

But to push to the local container registry we need the handle to the docker socket of the host which is usually the file `/var/run/docker.sock`. We add the handle in our pipeline using the `volume` mounts. As we are mounting the host file into the container its expected to instruct Drone to run in **trusted** mode to add extra privileges to the step containers.

Build Application
-----------------

Let us try running the application without `trusted`,

```shell
drone exec
```

The command should fail with the following output,

```text
linter: untrusted repositories cannot mount host volumes
```

Build and push the image by running the command,

```shell
drone exec --trusted
```

Once the pipeline is successful, you can test the built image by running the following command,

```shell
docker run --detach --name=hello-go -p8080:8080 localhost:5001/example/go-hello-world
```

Now doing a curl should return a response **Hello, World!**,

```shell
curl localhost:8080/hello-world
```

> **Plugins**:
>
> When we built the container image we used image called `plugins/docker` with custom settings what is that? It is a **Drone Plugin**.
>
>Plugins are pre-defined set of commands which is packed as a container image. Plugins are reusable and the parameters to the plugin commands are passed using *settings* block.
>
> Please check [Drone Plugins](https://plugins.drone.io) for a list of available plugins.

Including and Excluding Steps
=============================

Drone allows you to include and exclude steps of the pipeline at execution time.

Open the exercise folder **greeting** on the **Editor** tab and analyse the Drone pipeline. It is simple greeter application that says **hello world** in multiple languages.

```yaml
kind: pipeline
type: docker
name: default

steps:
  - name: english
    image: alpine
    commands:
      - echo hello world

  - name: french
    image: alpine
    commands:
      - echo bonjour monde

  - name: hindi
    image: alpine
    commands:
      - echo नमस्ते दुनिया

  - name: japanese
    image: alpine
    commands:
      - echo こんにちは世界
```

On the **Terminal** tab, navigate to `$TUTORIAL_HOME/greeting`,

```shell
cd $TUTORIAL_HOME/greeting
```

Include Steps
-------------

Let us assume that we want to greet in *English* and *French* i.e. run only the steps **english** and **french**. To do that run the following command,

```shell
drone exec --include=english --include=french
```

A successful run will shown an output like,

```text
[english:0] + echo hello world
[english:1] hello world
[french:0] + echo bonjour monde
[french:1] bonjour monde
```

Exclude Steps
-------------

Excluding steps works in a similar way as that of include but with exclude we specify the steps to be excluded. Say we want to exclude **english** and **french** steps from our pipeline run, to do that run the `drone` command as shown,

```shell
drone exec --exclude=english --exclude=french
```

A successful run will shown an output like,

```text
[hindi:0] + echo नमस्ते दुनिया
[hindi:1] नमस्ते दुनिया
[japanese:0] + echo こんにちは世界
[japanese:1] こんにちは世界
```

Injecting Environment Variables
===============================

In many cases we might need to load few settings from environment variables or files, `drone` provides an option called `--env-file` that allows to load the environment variable(s) which can then be used within step containers.

Open the exercise folder **env-greeting** on the **Editor**.

Let us analyse the pipeline file by opening it using the **Editor** tab,

```yaml
kind: pipeline
type: docker
name: default

steps:
  - name: english
    image: alpine
    commands:
      - echo $GREETING_MESSAGE
```

Loading Environment Variables
-----------------------------

As you notice the pipeline step *english* prints the value `${GREETING_MESSAGE}`. The variable `${GREETING_MESSAGE}` loaded from the file `my-env`.

```yaml title="my-env"
GREETING_MESSAGE=Hello, World!
```

Run Pipeline
------------

In order for the steps to use `${GREETING_MESSAGE}` as environment variable, we use the `--env-file` option of the `drone` command to load `my-env` into the step container.

On the **Terminal** tab, navigate to `$TUTORIAL_HOME/use-env`

```shell
cd $TUTORIAL_HOME/env-greeting
```

Now run the pipeline,

```shell
drone exec --env-file=my-env
```

The pipeline execution will show an output like,

```text
[english:0] + echo $GREETING_MESSAGE
[english:1] Hello, World!
```

Injecting Secrets
=================

Drone also allows to load few settings from secrets, `drone` provides an option called `--secret-file` with which we can load the secrets as environment variables and use them in our steps.

> **NOTE:**
> Secrets are masked in pipeline logs

Let us analyse the `using-secrets` pipeline file using the **Editor** tab,

```yaml
kind: pipeline
type: docker
name: default

steps:
  - name: what is today?
    image: postgres:14.4-alpine
    commands:
      - sleep 10
      - psql --host=$PGHOST --port=$PGPORT -w -c 'SELECT NOW() as TODAY;'
    environment:
      PGHOST: postgres
      PGPORT: 5432
      PGDATABASE:
        from_secret: postgres_db
      PGUSER:
        from_secret: postgres_user
      PGPASSWORD:
        from_secret: postgres_password

services:
  - name: postgres
    image: postgres:14.4-alpine
    environment:
      POSTGRES_DB:
        from_secret: postgres_db
      POSTGRES_USER:
        from_secret: postgres_user
      POSTGRES_PASSWORD:
        from_secret: postgres_password
```

As you have noted from the highlighted line, we inject the environment variable called `PGDATABASE` with value from secret `postgres_db` using a special `drone` attribute named **from_secret**.The **from_secret** attribute value maps to the secret key that gets loaded using `--secret-file` option.

Loading Secrets
---------------

As you notice the pipeline step *what is today?* runs a `psql` command that selects the current date using the SQL query `SELECT NOW()`.

To `psql` command expects the environment variables:

- PGHOST
- PGPORT
- PGDATABASE
- PGUSER
- PGPASSWORD

We define *PGHOST*,*PGPORT* as plain text values in the **environment** block,  whereas *PGDATABASE*,*PGUSER*,*PGPASSWORD* are loaded from the secret file `secret.txt`.

```yaml title="secret.txt"
postgres_db=demodb
postgres_user=postgress
postgres_password=pa55Word!
```

Run Pipeline
------------

On the **Terminal** tab, navigate to `$TUTORIAL_HOME/using-secrets`

```shell
cd $TUTORIAL_HOME/using-secrets
```

Now run the pipeline,

```shell
drone exec --secret-file=secret.txt
```

The pipeline execution will show an output like ( output trimmed for brevity),

```text
...
[what is today?:1] + psql --host=$PGHOST --port=$PGPORT -w -c 'SELECT NOW() as TODAY;'
[what is today?:2]              today
[what is today?:3] -------------------------------
[what is today?:4]  2022-07-19 06:48:05.497789+00
[what is today?:5] (1 row)
[what is today?:6]
...
```

> **IMPORTANT:**
>
> For this demo we add the secret.txt git, but for production  scenarios make sure the file name or pattern is added to `.gitingore`
>
> As you noticed we used new configuration block in the pipeline called *services*. Services are detached containers that are not part of your step but will be used by steps. These containers are killed automatically after the pipeline execution ends. Be sure to check the [Drone Docs](https://docs.drone.io) for more information about **services**.

Multiple Pipelines or Stages
============================

A `drone` pipeline `.drone.yml` can have multiple pipelines a.k.a **stages**. Each stage is its own YAML document. The `drone` command has an option called `--pipeline` that allows you to invoke the required stage,

Open the examples folder `multiple-stages` in the **Editor** tab,

As you see form the pipeline file, it has four stages namely *english,french,hindi and japanese*.

```yaml
kind: pipeline
type: docker
name: english

steps:
  - name: greeting
    image: alpine
    commands:
      - echo hello world
---
kind: pipeline
type: docker
name: french

steps:
  - name: greeting
    image: alpine
    commands:
      - echo bonjour monde
---
kind: pipeline
type: docker
name: hindi

steps:
  - name: greeting
    image: alpine
    commands:
      - echo नमस्ते दुनिया

---
kind: pipeline
type: docker
name: japanese

steps:
  - name: greeting
    image: alpine
    commands:
      - echo こんにちは世界
```

Run a Stage
-----------

On the **Terminal** tab, navigate to `$TUTORIAL_HOME/multiple-stages`

```shell
cd $TUTORIAL_HOME/multiple-stages
```

Let us say you want to run the *japanese* stage of the pipeline,

```shell
drone exec --pipeline=japanese
```

The command should shown an output like,

```text
[greeting:0] + echo こんにちは世界
[greeting:1] こんにちは世界
```

Exchange Information Between Steps
==================================

The pipeline does not run independently, each step produces some output that may be used by the next step(s). The easiest way to exchange information between steps is using Volume mounts,

Open thr exercise folder `$TUTORIAL_HOME/exchange-info` on the editor **Editor**,

Let us check the pipeline file,

```yaml
kind: pipeline
type: docker
name: default

steps:
  - name: store-message
    image: busybox
    commands:
      - echo 'Hello, World!' > /data/message.txt
    volumes:
     - name: data-vol
       path: /data
  - name: show-message
    image: busybox
    commands:
      - cat /data/message.txt
    volumes:
     - name: data-vol
       path: /data
volumes:
  - name: data-vol
    temp: {}
```

As you noticed both the steps *store-message* and *show-message* mounts a Volume on same path `/data`. The *store-message* writes an output to the file `/data/message.txt` which is then read and displayed by the next step *show-message*.

On the **Terminal** tab, navigate to `$TUTORIAL_HOME/exchange-info`

```shell
cd $TUTORIAL_HOME/exchange-info
```

Run the pipeline to see it in action,

```shell
drone exec
```

A successful pipeline execution will show the following output,

```text
[store-message:0] + echo 'Hello, World!' > /data/message.txt
[show-message:0] + cat /data/message.txt
[show-message:1] Hello, World!
```

🏁 Finish
=========

To complete this challenge, press **Check**.
