---
title: Automatically configure Telegraf
seotitle: Automatically configure Telegraf for InfluxDB v2.1
description: >
  Use the InfluxDB UI to automatically generate a Telegraf configuration,
  then start Telegraf using the generated configuration file.
aliases:
  - /influxdb/v2.1/collect-data/use-telegraf/auto-config
  - /influxdb/v2.1/write-data/use-telegraf/auto-config
menu:
  influxdb_2_1:
    parent: Telegraf (agent)
weight: 201
related:
  - /influxdb/v2.1/telegraf-configs/create/
---

The InfluxDB user interface (UI) can automatically create Telegraf configuration files based on user-selected Telegraf plugins.
This article describes how to create a Telegraf configuration in the InfluxDB UI and
start Telegraf using the generated configuration file.

{{< youtube M8KP7FAb2L0 >}}

{{% note %}}
_View the [requirements](/influxdb/v2.1/write-data/no-code/use-telegraf#requirements)
for using Telegraf with InfluxDB v2.1._
{{% /note %}}

## Create a Telegraf configuration

{{% oss-only %}}1. Open the InfluxDB UI _(default: [localhost:8086](http://localhost:8086))_.{{% /oss-only %}} 
{{% cloud-only %}}1. Open the InfluxDB Cloud UI.{{% /cloud-only %}}
2. In the navigation menu on the left, select **Load Data** > **Telegraf**.

    {{< nav-icon "load data" >}}

4. Click **{{< icon "plus" >}} {{< caps >}}Create Configuration{{< /caps >}}**.
5. In the **Bucket** dropdown, select the bucket where Telegraf will store collected data.
6. Select one of the available plugins and click **{{< caps >}}Continue Configuring{{< /caps >}}**.
8. Provide a **Configuration Name** and an optional **Configuration Description**.
9. Click **{{< caps >}}Save and test{{< /caps >}}**.
10. The **Test Your Configuration** page provides instructions for how to start Telegraf using the generated configuration.
   _See [Start Telegraf](#start-telegraf) below for detailed information about what each step does._
11. Once Telegraf is running, click **{{< caps >}}Listen for Data{{< /caps >}}** to confirm Telegraf is successfully sending data to InfluxDB.
Once confirmed, a **{{< caps >}}Connection Found!{{< /caps >}}** message appears.
12. Click **{{< caps >}}Finish{{< /caps >}}**. Your Telegraf configuration name and the associated bucket name appear in the list of Telegraf configurations.


### Windows

If you plan to monitor a Windows host using the System plugin, you must complete the following steps.

1. In the list of Telegraf configurations, double-click your
    Telegraf configuration, and then click **{{< caps >}}Download Config{{< /caps >}}**.
2. Open the downloaded Telegraf configuration file and replace the `[[inputs.processes]]` plugin with one of the following Windows plugins, depending on your Windows configuration:

   - [`[[inputs.win_perf_counters]]`](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/win_perf_counters)
   -  [`[[inputs.win_services]]`](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/win_services)

3. Save the file and place it in a directory that **telegraf.exe** can access.


## Start Telegraf

Requests to the [InfluxDB v2 API](/influxdb/v2.1/reference/api/) must include an API token.
A token identifies specific permissions to the InfluxDB instance.

### Configure your token as an environment variable

1. Find your API token. _For information about viewing tokens, see [View tokens](/influxdb/v2.1/security/tokens/view-tokens/)._

2. To configure your API token as the `INFLUX_TOKEN` environment variable, run the command appropriate for your operating system and command-line tool:

{{< tabs-wrapper >}}
{{% tabs %}}
[macOS or Linux](#)
[Windows](#)
{{% /tabs %}}

{{% tab-content %}}
```sh
export INFLUX_TOKEN=YourAuthenticationToken
```
{{% /tab-content %}}

{{% tab-content %}}

{{< code-tabs-wrapper >}}
{{% code-tabs %}}
[PowerShell](#)
[CMD](#)
{{% /code-tabs %}}

{{% code-tab-content %}}
```sh
$env:INFLUX_TOKEN = "YourAuthenticationToken"
```
{{% /code-tab-content %}}

{{% code-tab-content %}}
```sh
set INFLUX_TOKEN=YourAuthenticationToken
# Make sure to include a space character at the end of this command.
```
{{% /code-tab-content %}}
{{< /code-tabs-wrapper >}}

{{% /tab-content %}}
{{< /tabs-wrapper >}}

### Start the Telegraf service

Start the Telegraf service using the `-config` flag to specify the location of the generated Telegraf configuration file.

- For Windows, the location is always a local file path.
- For Linux and macOS, the location can be a local file path or URL.

Telegraf starts using the Telegraf configuration pulled from InfluxDB API.

{{% note %}}
InfluxDB host URLs and ports differ between InfluxDB OSS and InfluxDB Cloud.
For the exact command, see the Telegraf configuration **Setup Instructions** in the InfluxDB UI.
{{% /note %}}

```sh
telegraf -config http://localhost:8086/api/v2/telegrafs/0xoX00oOx0xoX00o
```

## Manage Telegraf configurations

For more information about managing Telegraf configurations in InfluxDB, see
[Telegraf configurations](/influxdb/v2.1/telegraf-configs/).
