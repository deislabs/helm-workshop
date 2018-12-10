# Adding the Redis and Postgres Databases

In the previous section we added the first service. Now we need to add our data layer. We could create charts from scratch, but instead, let's use some existing ones.

Try this:

```console
$ helm search redis
NAME                                    CHART VERSION   APP VERSION     DESCRIPTION
stable/prometheus-redis-exporter        0.3.0           0.16.0          Prometheus exporter for Redis metrics
stable/redis                            3.7.6           4.0.11          Open source, advanced key-value store. It is of...
stable/redis-ha                         2.2.1           4.0.8-r0        Highly available Redis cluster with multiple se...
stable/sensu                            0.2.3           0.28            Sensu monitoring framework backed by the Redis ...
```

See how we already have a chart that adds `redis` support? Rather than create our own, let's use that. (Note: If you are feeling extra ambitious, you can write your own Redis chart. But it will take about 45 minutes.)

## Subcharts and `requirements.yaml`

In Helm, a _subchart_ is a regular Helm chart that is a dependency of your chart. By adding it to a special file called `requirements.yaml`, you can let Helm manage that chart for you.

Let's create a `requirements.yaml` file next to the `Chart.yaml` in our chart. Here's what belongs inside:

```yaml
dependencies:
  - name: redis        # from search results above
    version: 3.7.6     # Also from the search results above
    repository: https://kubernetes-charts.storage.googleapis.com
```

Note that the `repository` URL comes from running `helm repo list` and copying the URL for the `stable` repo, which is where the Redis chart lives.

### Add PosgreSQL on your own

Next, run `helm search postgresql` and repeat the above for Postgres. By the end of that step, you should have two items in your `dependencies` array in `requirements.yaml`

## Download the Dependencies

Run `helm dep up` to have Helm fetch those dependencies for you. Why you are done with that command, you should be able to see two new directories inside of your chart's `charts` subdirectory.

## Upgrade Your Release

If you now run `helm upgrade voting-app .` you can upgrade your chart and get the new redis and postgres databases installed.

> Note: If you are on Azure, make sure to add the `--set service.type=loadbalancer` flag again

## Tip: Re-installing

If you ever need to reinstall instead of upgrading, simply run the following:

```
$ helm delete --purge voting-app
```

Then you can re-run the installer freshly.


