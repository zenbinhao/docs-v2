---
title: Manage TSM Data
seotitle: Manage TSM in InfluxDB
description: 
menu:
  influxdb_2_0:
    parent: Administration
weight: 100
influxdb/v2.0/tags: [administration]
---

## Run TSM report

influxd inspect report-tsm

## Check the consistency of TSM files

 `influxd inspect verify-tsm`        Verifies the integrity of TSM files

## Output TSM data from WAL files

`influxd inspect dump-wal`

<!-- ### Export block data -->

## Check for corrupt WAL files

`influxd inspect verify-wal`
