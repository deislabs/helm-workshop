# Helm Workshop: Microsoft Reactor, 2018

The purpose of this workshop is to lean how to build new Helm 2 charts.

## Getting Familiar with Helm

If this is your first time using Helm, start by reading through the [introduction to Helm](https://docs.helm.sh).

Installing Helm is also covered there. On Mac, the easiest way to install Helm is to use Homebrew:

```console
$ brew install helm
```

## Charts

Helm uses charts as the installable unit. A Chart is a package containing at least three things:

- A [`Chart.yaml`](https://docs.helm.sh/developing_charts#the-chart-yaml-file), which describes the chart
- [Templates](https://docs.helm.sh/developing_charts#template-files), which Helm transforms into Kubernetes manifests
- A [`values.yaml` file](https://docs.helm.sh/developing_charts#templates-and-values), which defines and describes configurable parameters

These files are all bundled together into an archive that can easily be moved from one place to another. Charts are frequently stored in _chart repositories_ like https://hub.helm.sh. Helm repositories contain collections of charts that Helm can inspect, search, and install.

## Goal

Our goal for this workshop is to create a Helm chart that takes Docker's popular voting app demo and installs it in a Kubernetes cluster.

We will start with a simple chart, and then add from there.

Our application will have five parts:

1. A front-end for users to vote
2. A back-end that tallies votes
3. An admin interface to see the results
4. A Redis cache
5. A database

### Quick Reference

In this guide, we will be building a chart based on a sample voting app. Here are the images you will need:

  - `redis:alpine`
  - `microsoft/mssql-server-linux:latest`

**FIXME:** Need the images for these three:

```
    build: ./voting-app/.
    build: ./worker
    build: ./result-app/.
```

## Creating a Chart

To get started, we need to create a new directory. In this guide, we will be using linux filepath conventions, though Windows-style filepaths work with very similar commands.

Start by running the Helm command to create a new basic chart:

```console
$ helm create voter
```

This will create a new directory named `voter`, and scaffold it out as follows:

```text
voter/
├── Chart.yaml
├── charts
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── ingress.yaml
│   └── service.yaml
└── values.yaml
```

We will start by modifying the `Chart.yaml`

## Chart.yaml

All we need to change in this file is the description.

```yaml
apiVersion: v1      # The chart schema version, always v1 for Helm 2
version: 0.1.0      # The version of the chart. Change this for each release.
appVersion: "1.0"   # The version of the main app in this chart. OPTIONAL
description: An example cloud native voting app
name: voter
```

Save this. You can verify that you correctly entered the data by running:

```console
$ helm inspect chart ./voter
apiVersion: v1
appVersion: "1.0"
description: An example cloud native voting app
name: voter
version: 0.1.0
```

Next, we'll start working with Helm templates

## Configuring the Front-End

Take a look at the `values.yaml` file generated for us:

```yaml
# Default values for voter.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: nginx
  tag: stable
  pullPolicy: IfNotPresent

nameOverride: ""
fullnameOverride: ""

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  path: /
  hosts:
    - chart-example.local
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #  cpu: 100m
  #  memory: 128Mi
  # requests:
  #  cpu: 100m
  #  memory: 128Mi

nodeSelector: {}

tolerations: []

affinity: {}

```

This default values file has a number of standard configurable parameters pre-defined. The parameters here are used by the files in `templates`.

The default Helm chart defines three Kubernetes resources:

- A Deployment for running a microservice
- A Service for routing traffic to that microservice
- An Ingress that can optionally be turn on to allow external traffic to access your app

It just so happens that we need this things for our example. So to get started, we'll just change the default values to make use of them.

In the `images` section, make the following changes:

- Point `repository` to `technosophos/voting-app`
- Set `tag` to `1.0.0`

Once you've done that, you will have a runnable chart that we can test.

## Adding the Back-End

Helm templates are written in a language called `gotemplate`. Along with the basic functions provided by the core language, Helm also provides dozens of extra functions from the [Sprig library](https://github.com/Masterminds/sprig).

The best reference for Helm chart syntax, style, and techniques can be found in the official [Helm chart guide](https://docs.helm.sh/developing_charts/#charts).
