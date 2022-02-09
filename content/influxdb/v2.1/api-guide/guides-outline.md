1. Introduction
    This guide is for developers who want to build Internet-of-Things (IoT) applications using the InfluxDB API.
    The InfluxDB platform provides a REST API for writing, querying, and managing time series data.
    API client libraries maintained by InfluxDB and the user community enable developers to take
    advantage of common patterns in their language of choice and avoid writing boilerplate code.
    In this guide, you'll walk through the process of building a real application that manages IoT devices, writes data to InfluxDB, queries data from InfluxDB to create data visualizations, and monitors the health of devices and the application itself.
    You'll learn:
    - the basics of InfluxDB and the REST API
    - how to install a client library
    - how to write data to influxDB
    - how to query data
1. Setup InfluxDB
   1. InfluxDB URL, Org, Bucket, All-Access Token
1. Start with an API client (InfluxDB basics)
   1. API basics
   2. Create an API token
   3. Create a bucket
      1. Measurements, time series
      2. Measurement schemas (aka bucket schemas)
   1. Install a client library
   4. Write data
      1. Line protocol
   5. Query InfluxDB
      1. Flux
   6. Aggregate and downsample your data
1. IoT Dev Center
   1. Architecture
      - IoT Dev Center serves a modern React UI and a server-side API (/api/env/<deviceID>).
   2. Simulate devices
      - In IoT Dev Center, you can register __virtual devices__ that simulate real IoT devices and demonstrate the IoT Dev Center app before advancing to physical devices and other clients.
1. Register an IoT device.
   - On "Device Registrations" load, ...
     - If the device is registered, i.e. it has an authorization in InfluxDB, then IoT Dev Center API returns authorization and InfluxDB configuration properties as configuration details for the device.
            [Example]
   - On "Register", ...
     - If the device is not registered, IoT Dev Center API sends a request to the
        InfluxDB API `/api/v2/authorizations` endpoint to create an API token with the name `IoT Center: DEVICE_ID`. The new token will be used in future API requests and token management (e.g. expiration and rotation).
     - IoT Dev Center app uses the `IoT Center: ` prefix to identify and list registered devices.
   - On "Delete", ...
   - On "Device Settings"
     - Device configuration details are composed of your InfluxDB configuration and authorization details for the Device ID.
  ## Authorization and authentication in InfluxDB
   IoT devices generate measurement data....
   To write time series measurements to InfluxDB, your application or device needs to be authorized.
   An InfluxDB **authorization** consists of a set of permissions and an API token.
   To authenticate InfluxDB API requests, the device passes the API token in the `Authorization` request header.
2. Device dashboard

1. Manage and secure tokens
   1. InfluxDB Cloud (token access restrictions)
1. Collect IoT data
   1. Data segmentation strategies
1. Query data
2. Monitor the IoT app
2. Tasks
2. Dashboards
3. Monitor IoT device data


IoT Center SUMMARY.md:
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
