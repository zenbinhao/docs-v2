1. Introduction
    This guide is for developers who want to build Internet-of-Things (IoT) applications using the InfluxDB API.
    The InfluxDB platform provides a REST API for writing, querying, and managing time series data.
    API client libraries maintained by InfluxDB and the user community enable developers to take
    advantage of common patterns in their language of choice and avoid writing boilerplate code.
    In this guide, you'll walk through the process of building a real application that manages IoT devices, writes data to InfluxDB, queries data from InfluxDB to create data visualizations, and monitors the health of devices and the application itself.
    You'll learn:
    - the basics of InfluxDB
    - how to install a client library
    - how to use the InfluxDB API
    - how to write data to influxDB
    - how to query data
1. Setup InfluxDB
   1. InfluxDB URL, Org, Bucket, All-Access Token
1. Start with an API client (InfluxDB basics)
   1. Install a client library
   2. Create an API token
   3. Create a bucket
      1. Measurements, timeseries
      2. Measurement schemas (aka bucket schemas)
   4. Write data
      1. Line protocol
   5. Query influxDB
      1. Flux
   6. Aggregate and downsample your data
1. Register an IoT device.
   ## Authorization and authentication in InfluxDB
   IoT devices generate measurement data....
   To write timeseries measurements to InfluxDB, your application or device needs to be authorized.
   An InfluxDB **authorization** consists of a set of permissions and an API token.
   To authenticate InfluxDB API requests, the device passes the API token in the `Authorization` request header.
   1. IoT Dev Center app serves an API (/api/env/<deviceID>).
   2. If the device is registered, IoT Dev Center API returns device details.
      If the device is not registered, IoT Dev Center API sends a request to the
      InfluxDB API `/api/v2/authorizations` endpoint to create an authorization.
   3. IoT Dev Center app stores the device configuration and the associated new token to be used in future API requests and token management (e.g. expiration and rotation).
      - See the IoT Dev Center example
1. Manage and secure tokens
   1. InfluxDB Cloud (token access restrictions)
1. Collect IoT data
   1. Data segmentation strategies
1. Query data
2. Monitor the IoT app
2. Tasks
2. Dashboards
3. Monitor


IoT Center SUMMARY.md
- [Introduction](./introduction.md)

- [IoT Center: overview](./iot-center.md)
  - [Architecture](./architecture.md)
  - [Getting Started: InfluxDB](./getting-started-influxdb.md)
  - [Getting Started: IoT Center](./getting-started.md)

- [Create the server: *A Tale of Two APIs*](./iot-center-api.md)
  - [Configure InfluxDB](./api/influxdb-setup.md)
  - [InfluxDB API: Set up and InfluxDB instance](./api/onboarding.md)
    <!-- Maybe at this point we actually skip to creating the virtual device to show the write API? -->
  - [IoT Center API: Register and authorize devices](./api/authorization.md)
  - [Serve the API](./api/serving-the-api.md)

- [Create the UI](./iot-center-ui.md)
  - [Visualize data](./ui/dashboard.md)
  - [Manage devices](./ui/device-registration.md)
  - [Write data](./ui/virtual-device.md)

- [Real-world IoT devices](./devices.md)
  - [Raspberry Pi Python client](./device-code/client_python.md)
  <!-- - [Arduino client](./device-code/client_arduino.md) -->

---

[Further reading](./further-reading.md)
