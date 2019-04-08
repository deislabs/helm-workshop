# Templating Charts

Helm charts contain templates, which at its core are resource descriptions that Kubernetes can understand. The Helm template language is [written in `gotemplate`](https://golang.org/pkg/text/template/), a templating language provided by the Go standard library. Templates can be rendered locally for debugging purposes using the `helm template` command.

This guide will teach you the ins and outs of the Helm template language.

## Getting Started with Chart Templates

In the last section, we created a Helm chart that contained three templates:

- A Deployment for running a microservice (`deployment.yaml`)
- A Service for routing traffic to that microservice (`service.yaml`)
- An Ingress that can optionally be turn on to allow external traffic to access your app (`ingress.yaml`)

Let's start by diving into the template language.

## About `gotemplate`

As mentioned before, Helm templates are [written in `gotemplate`](https://golang.org/pkg/text/template/), a templating language provided by the Go standard library. Along with the basic functions provided by `gotemplate`, Helm provides dozens of extra functions from the [Sprig library](https://github.com/Masterminds/sprig).

The output of a template MUST be a YAML-formatted resource description that Kubernetes can understand, such as a Deployment or a Service.

Let's take a look at a snippet of the Deployment template created in an earlier section:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ template "voter.fullname" . }}
  labels:
    app: {{ template "voter.name" . }}
    chart: {{ template "voter.chart" . }}
    release: {{ .Release.Name }}
    heritage: {{ .Release.Service }}
spec:
  replicas: {{ .Values.replicaCount }}
```

As we can see, this looks a bit like a Deployment object in YAML format, sprinked with a few templates from `gotemplate`. We can see a few unique templates. Let's look at a few in particular:

1. `{{ template "voter.fullname" . }}`
1. `{{ template "voter.name" . }}`
1. `{{ .Release.Name }}`
1. `{{ .Values.replicaCount }}`

`template "voter.fullname"` is a template partial. Template partials are "helper" functions defined within the chart to abstract some common funcion calls to make the templates more easily readable and DRY. When we first created the chart, a file called `_helpers.tpl` was created in the `templates` directory. That file is the default location for template partials, and Helm provides a few template partials out the gate in that file.

`"voter.fullname"` accepts one variable as input: the dot variable (`.`). The dot variable contains all of the variables injected by Helm into the template engine, such as values or information about the cluster. More on the dot variable in a second.

The output of `"voter.fullname"` is the chart name (voter) and the release name (my-test), joined with a hyphen (voter-my-test).

`"voter.name"` is just the chart name, so "voter".

To explain `.Release.Name` and `.Values.replicaCount`, we'll have to explain the dot variable in more depth.

### The Dot Variable and Built-in Objects

When a Helm chart is being rendered, objects are passed into the template from the rendering engine. Objects can be simple and have just one value, or they can contain other objects or functions. For example, the `.Release` object contains several objects (like `.Release.Name`), and the `.Files` object has a few functions to help grab the contents of a file in the chart.

In the previous section, we used `{{ .Release.Name }}`, which is the name of the release we passed on the CLI to `helm install`. `.Release` is one of the top-level objects that you can access in your templates.

There are a few other built-in objects provided by Helm. We'll see more of these in the templates:

- `Release`: This object describes the release itself. It has several objects inside of it:
  - `Release.Name`: The release name
  - `Release.Time`: The time of the release
  - `Release.Namespace`: The namespace to be released into (if the manifest doesn't override)
  - `Release.Service`: The name of the releasing service (always `Tiller`).
  - `Release.Revision`: The revision number of this release. It begins at 1 and is incremented for each `helm upgrade`.
  - `Release.IsUpgrade`: This is set to `true` if the current operation is an upgrade or rollback.
  - `Release.IsInstall`: This is set to `true` if the current operation is an install.
- `Values`: Values passed into the template from the `values.yaml` file and from user-supplied files. By default, `Values` is empty.
- `Chart`: The contents of the `Chart.yaml` file. Any data in `Chart.yaml` will be accessible here. For example `{{.Chart.Name}}-{{.Chart.Version}}` will print out the `mychart-0.1.0`.
  - The available fields are listed in the [Charts Guide](https://github.com/helm/helm/blob/master/docs/charts.md#the-chartyaml-file)
- `Files`: This provides access to all non-special files in a chart. While you cannot use it to access templates, you can use it to access other files in the chart. See the section _Accessing Files_ for more.
  - `Files.Get` is a function for getting a file by name (`.Files.Get config.ini`)
  - `Files.GetBytes` is a function for getting the contents of a file as an array of bytes instead of as a string. This is useful for things like images.
- `Capabilities`: This provides information about what capabilities the Kubernetes cluster supports.
  - `Capabilities.APIVersions` is a set of versions.
  - `Capabilities.APIVersions.Has $version` indicates whether a version (`batch/v1`) is enabled on the cluster.
  - `Capabilities.KubeVersion` provides a way to look up the Kubernetes version. It has the following values: `Major`, `Minor`, `GitVersion`, `GitCommit`, `GitTreeState`, `BuildDate`, `GoVersion`, `Compiler`, and `Platform`.
  - `Capabilities.TillerVersion` provides a way to look up the Tiller version. It has the following values: `SemVer`, `GitCommit`, and `GitTreeState`.
- `Template`: Contains information about the current template that is being executed
  - `Name`: A namespaced filepath to the current template (e.g. `mychart/templates/mytemplate.yaml`)
  - `BasePath`: The namespaced path to the templates directory of the current chart (e.g. `mychart/templates`).

When you provide the following in `values.yaml`:

```yaml
image:
    repository: technosophos/voting-app
```

Helm will render that as `.Values.image.repository` to be made available to you in the templates.

## Learning more about Chart Templates

Run `helm template ./voter` to view the chart fully rendered as YAML manifests. This is what will eventually be supplied to Kubernetes.

Go through the rest of the templates and [The Chart Developer's Guide](https://docs.helm.sh/chart_template_guide/#the-chart-template-developer-s-guide). Experiment by adding and removing fields, or by modifying existing ones. Also try and change some values using the --set flag, like `helm template ./voter --set image.repository=foo`.

Try and modify the Deployment object. What happens to the chart when you run `helm template ./voter` now?

## Adding Templates for the Voter App Backend

In the previous sections, we set up the voter app frontend and the two databases. Now we can add the backend. This will make our chart fully functional.

Copy this YAML file into your `templates/` directory, and then transform it into a template.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: worker
  labels:
    app: worker
spec:
  replicas: 1
  selector:
    matchLabels:
      app: worker
  template:
    metadata:
      labels:
        app: worker
    spec:
      containers:
      - image: dockersamples/examplevotingapp_worker
        name: worker
```

Remember to make sure that the name is unique for each installation.

If you want to make a value configurable via the `values.yaml` file, simply set a default value in `values.yaml` and then access it here using `.Values.YOUR_NEW_NAME`.

Once your template is done, re-install or upgrade your Helm chart. At this point you should be able to vote for your favorite animal!

## Add the Results Viewer

Ready for more? There's one last piece of this app: The results viewer. To add this part, we need to create
two more template files:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: result
  labels:
    app: result
spec:
  replicas: 1
  selector:
    matchLabels:
      app: result
  template:
    metadata:
      labels:
        app: result
    spec:
      containers:
      - image: dockersamples/examplevotingapp_result:before
        name: result

---

apiVersion: v1
kind: Service
metadata:
  name: result
spec:
  type: NodePort
  ports:
  - name: "result-service"
    port: 5001
    targetPort: 80
  selector:
    app: result
```

Make sure you configure the service for outside access the way you configured the voting app frontend. (Take a look at your `values.yaml` to make sure)

The voting app also requires a `db` and a `redis` service name, so let's create those:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
  labels:
    app: redis
    chart: redis-3.7.6
    release: {{ .Release.Name }}
    heritage: "Tiller"
  annotations:
spec:
  type: ClusterIP
  ports:
  - name: redis
    port: 6379
    targetPort: redis
  selector:
    app: redis
    release: {{ .Release.Name }}
    role: master
```

```yaml
# Source: voter/charts/postgresql/templates/svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: db
  labels:
    app: db
    chart: postgresql-2.4.0
    release: {{ .Release.Name }}
    heritage: "Tiller"
spec:
  type: ClusterIP
  ports:
  - name: postgresql
    port:  5432
    targetPort: postgresql
  selector:
    app: db
    release: {{ .Release.Name }}
    role: master
```

Congratulations! You've just transformed a five-component multi-tier microservice buzzword-laden app into a Helm chart!
