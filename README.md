# Helm Workshop: Microsoft Reactor, 2018

The purpose of this workshop is to lean how to build new Helm 2 charts.

## Workshop Format
This workshop is split into numbered sections outlined below:
1. [Getting Started](1-getting-started/)
2. [Charts](2-charts/)

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

## Adding the Back-End

Helm templates are written in a language called `gotemplate`. Along with the basic functions provided by the core language, Helm also provides dozens of extra functions from the [Sprig library](https://github.com/Masterminds/sprig).

The best reference for Helm chart syntax, style, and techniques can be found in the official [Helm chart guide](https://docs.helm.sh/developing_charts/#charts).
