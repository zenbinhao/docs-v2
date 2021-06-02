---
title: Kubernetes Operator
description:
menu:
  telegraf_1_18:
    name: Kubernetes operator
    weight: 20
    parent: Administration
---

https://github.com/influxdata/telegraf-operator

Use the Telegraf operator to define a common output destination for all your metrics, and configure Sidecar monitoring on your application pods using labels.

The Telegraf operator expands Kubernetes to support other workflows and functionalities specific to an application. It packages the operational aspects for deploying a Telegraf agent on Kubernetes as an application Sidecar, and configures it to scrape the exposed application metrics. All defined declaratively in a yaml file.

## Install the Telegraf operator in Kubernetes

Install the Telegraf operator via `kubectl` using the command below:

`kubectl apply -f telegraf-operator.yml`


## Start scraping metrics

The Telegraf operator starts watching for pods being deployed with a specific set of pod annotations.

It will then take care of installing Telegraf Sidecars with the respective input plugin configuration to those pods automatically, and sending the metrics data to the output you’ve set up. Your users deploying applications never need to worry about configuring a metrics destination. You set it once for the entire cluster.

Example .yml file: https://github.com/influxdata/telegraf-operator/blob/master/deploy/dev.yml


### Example DaemonSet deployment yaml file with Telegraf configuration data

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: my-application
  namespace: default
spec:
  selector:
    matchLabels:|
      app:my-application
template:
  metadata:
    labels:
      app: my-application
    annotations:
      telegraf.influxdata.com/port: "8080"
      telegraf.influxdata.com/path: /v1/metrics
      telegraf.influxdata.com/interval: 5s
      telegraf.influxdata.com/scheme: http
      telegraf.influxdata.com/internal: "true"
  spec:
    containers:
      - name: my-application
        image: my-application:latest
```

### Example StatefulSet deployment of Redis yaml file with Telegraf configuration data

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
  namespace: test
spec:
  selector:
    matchLabels:
      app: redis
  serviceName: redis
template:
  metadata:
    labels:
      app: redis
    annotations:
      telegraf.influxdata.com/inputs: |+
        [[inputs.redis]]
          servers = ["tcp://localhost:6379"]
        telegraf.influxdata.com/class: app
  spec:
    containers:
      - name: redis
        image: redis:alpine
```
