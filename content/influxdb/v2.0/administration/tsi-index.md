---
title: Manage TSI index
seotitle: Manage TSI index in InfluxDB
description:
menu:
  influxdb_2_0:
    parent: Administration
weight: 100
influxdb/v2.0/tags: [administration]
---

<!-- ## Rebuild the TSI index and series file -->

## Export TSI index data

`influxd inspect export-index` -h

This command will export all series in a TSI index to
SQL format for easier inspection and debugging.

Usage:
  influxd inspect export-index [flags]

Flags:
  -h, --help                 help for export-index
      --index-path string    Path to the index directory of the data engine
      --series-path string   Path to series file

## Report the cardinality of TSI files

`influxd inspect report-tsi` 

Reports the cardinality of TSI files

## Output low level TSI information

`influxd inspect dumptsi`

Dumps low-level details about tsi1 files.
