# Getting Started with Helm

Install helm on your machine and get familiar with the basic commands using the official [quickstart guide](https://docs.helm.sh/using_helm/#quickstart)

## Prerequisites

- A Kuberbetes cluster with kubectl configured to access it.

## Installing the Helm Client

On Mac, the easiest way to install Helm is to use Homebrew:

```console
$ brew install kubernetes-helm
```

On Ubuntu:

```console
$ sudo snap install helm --classic
```

And on Windows:

```console
$ choco install kubernetes-helm
```

> More installation methods documented in [installing helm](https://docs.helm.sh/using_helm/#installing-helm)

## Initialize Helm and Install Tiller

Once you have Helm ready, you can initialize the local environment and install Tiller into your Kubernetes cluster in one step.

> If your cluster has Role-Based Access Control (RAC) enabled, you may want to [configure a service account and rules](https://docs.helm.sh/using_helm/#role-based-access-control) before proceeding.

```console
$ helm init
```

Helm will use whatever Kubernetes cluster your context is pointing to. Use `kubectl` to learn about your contexts:

```console
$ kubectl config current-context
docker-for-desktop
```

You can verify your setup by running the version command.

```console
$ helm version
Client: &version.Version{SemVer:"v2.12.0", GitCommit:"d325d2a9c179b33af1a024cdb5a4472b6288016a", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.12.0", GitCommit:"d325d2a9c179b33af1a024cdb5a4472b6288016a", GitTreeState:"clean"}
```

Inspect the deployment if you get an error.

```console
$ kubectl -n kube-system describe deployment tiller-deploy
```

## Using Helm

If this is your first time using Helm, here are the critical commands.

- `helm help`: Show help. You can get more info on a command by doing `helm COMMAND --help`, such as `helm list --help`
- `helm search STRING`: Find charts to install
- `helm install -n NAME CHART`: Install something (create a release). Example: `helm install -n my-test stable/wordpress`
- `helm status NAME`: Get the status of a release
- `helm delete NAME`: Delete the release. Example: `helm delete my-test`


Up next: [Charts](../02-charts/).
