# Charts

Helm uses charts as the installable unit. A Chart is a package containing at least three components:

- A [`Chart.yaml`](https://docs.helm.sh/developing_charts#the-chart-yaml-file), which describes the chart
- [Templates](https://docs.helm.sh/developing_charts#template-files), which Helm transforms into Kubernetes manifests
- A [`values.yaml` file](https://docs.helm.sh/developing_charts#templates-and-values), which defines and describes configurable parameters

These files are all bundled together into an archive that can be moved from one place to another. Charts are frequently stored in _chart repositories_ like <https://hub.helm.sh>. Helm repositories contain collections of charts that Helm can inspect, search, and install.

To get more in depth information overview about charts, see this [Intro to Charts](https://docs.helm.sh/developing_charts/#charts) documentation.

## Creating a Chart

To get started, we need to create a new directory. In this guide, we will be using linux filepath conventions, though Windows-style filepaths work with similar commands.

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

This default values file has standard configurable parameters pre-defined. These parameters are used by files in `templates`.

The default Helm chart defines three Kubernetes resources:

- A Deployment for running a microservice
- A Service for routing traffic to that microservice
- An Ingress that can optionally be turn on to allow external traffic to access your app

It just so happens that we need these things for our example. So to get started, we'll change the default values to make use of them.

In the `images` section, make the following changes:

- Point `repository` to `dockersamples/examplevotingapp_vote`
- Set `tag` to `before`

And also change one thing in `service`:

- `port: 5000`

(See the images list in the main [README](../README.md) for a quick description of each.)

Once you've done that, you will have a runnable chart that we can test. We'll be able to install it, but not use it.

> If you are on Azure, add `--set service.type=LoadBalancer` to the end of the following command

```console
$ helm install -n voting-app ./voter
# ... some info
```

Note that both a deployment and a service are created for you. If you are using Docker Desktop, you'll be able to see your new app by pointing a web browser at `localhost:5000`. For other clusters, flag down one of us and ask about how to configure it.

At this point, your new app won't work, because it will be trying to contact a Redis database that is not there. In the next section, we'll look at adding some other parts.

Up next: [Subcharts](../03-subcharts/).
