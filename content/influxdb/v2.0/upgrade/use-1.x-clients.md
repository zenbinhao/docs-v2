---
title: Use 1.x client libraries with InfluxDB 2.0
description: >
  Use 1.x client libraries with InfluxDB 2.0 via the v1 compatibility API.
menu:
  influxdb_2_0:
    parent: Upgrade InfluxDB
    name: Use 1.x client libraries
weight: 11
---

InfluxDB OSS 2.0 provides a 1.x compatible authentication API that lets you
authenticate with a username and password like InfluxDB 1.x
_(separate from the credentials used to log into the InfluxDB user interface)_.

## Create authorization token

#### View existing v1 authorizations
Use the [`influx v1 auth list`](/influxdb/v2.0/reference/cli/influx/v1/auth/list/)
to list existing InfluxDB v1 compatible authorizations.

```sh
influx v1 auth list
```

#### Create a v1 authorization
Use the [`influx v1 auth create` command](/influxdb/v2.0/reference/cli/influx/v1/auth/create/)
to grant read/write permissions to specific buckets. Provide the following:

- [bucket IDs](/influxdb/v2.0/organizations/buckets/view-buckets/) to grant read
  or write permissions to
- new username
- new password _(when prompted)_

<!--  -->
```sh
influx v1 auth create \
  --read-bucket 00xX00o0X001 \
  --write-bucket 00xX00o0X001 \
  --username example-user
```
{{% /expand %}}
{{% expand "View and create InfluxDB DBRP mappings" %}}

## Create DBRP mapping

InfluxDB DBRP mappings associate database and retention policy combinations with
InfluxDB 2.0 [buckets](/influxdb/v2.0/reference/glossary/#bucket)

#### View existing DBRP mappings
Use the [`influx v1 dbrp list`](/influxdb/v2.0/reference/cli/influx/v1/dbrp/list/)
to list existing DBRP mappings.

```sh
influx v1 dbrp list
```

#### Create a DBRP mapping
Use the [`influx v1 dbrp create` command](/influxdb/v2.0/reference/cli/influx/v1/dbrp/create/)
command to create a DBRP mapping.
Provide the following:

- database name
- retention policy
- [bucket ID](/influxdb/v2.0/organizations/buckets/view-buckets/)

```sh
influx v1 dbrp create \
  --db example-db \
  --rp example-rp \
  --bucket-id 00xX00o0X001 \
  --default
```

_For more information about DBRP mapping, see [Database and retention policy mapping](/influxdb/v2.0/reference/api/influxdb-1x/dbrp/)._
{{% /expand %}}
{{< /expand-wrapper >}}
