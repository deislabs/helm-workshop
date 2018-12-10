## Getting Familiar with Helm

Install helm on your machine and get familiar with the basic commands using the official [quickstart guide](https://docs.helm.sh/using_helm/#quickstart)

On Mac, the easiest way to install Helm is to use Homebrew:

```console
$ brew install helm
```

On Ubuntu:

```console
$ sudo snap install helm --classic
```

And on Windows:

```console
$ choco install kubernetes-helm
```

If this is your first time using Helm, start by reading through the [introduction to Helm](https://docs.helm.sh). Alternative methods of installing Helm is also covered there.

## Initializing and Using Helm

Helm will use whatever Kubernetes cluster your context is pointing to. Use `kubectl` to learn about your contexts:

```console
$ kubectl config current-context
docker-for-desktop
```

Once Helm is installed, make sure to initialize it:

```console
$ helm init
```

If this is your first time using Helm, here are the critical commands:

- `helm help`: Show help. You can get more info on a command by doing `helm COMMAND --help`, such as `helm list --help`
- `helm search STRING`: Find things to install
- `helm install -n NAME CHART`: Install something (create a release). Example: `helm install -n my-test stable/wordpress`
- `helm status NAME`: Get the status of a release
- `helm delete NAME`: Delete the release. Example: `helm delete my-test`

