---
title: Prometheus input data format
description:
menu:
  telegraf_1_17:
    name: Prometheus
    weight: 10
    parent: Input data formats
---


The Prometheus input data format parses metrics in
[Prometheus Text-Based Format](https://prometheus.io/docs/instrumenting/exposition_formats/#text-based-format) into Telegraf metrics.

## Configuration

```toml
[[inputs.file]]
  files = ["example"]

  ## Data format to consume.
  ## Each data format has its own unique set of configuration options, read
  ## more about them here:
  ##   https://github.com/influxdata/telegraf/blob/master/docs/DATA_FORMATS_INPUT.md
  data_format = "prometheus"
```
