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


## Authorization and authentication in InfluxDB

     IoT devices generate measurement data....
     To write time series measurements to InfluxDB, your application or device needs to be authorized.
     An InfluxDB **authorization** consists of a set of permissions and an API token.
     To authenticate InfluxDB API requests, the device passes the API token in the `Authorization` request header.

See how to [create an authorization](/influxdb/v2.1/security/tokens/create-token/).

### IoT Center: device registrations

In "Device Registrations", the UI sends a request to the IoT Center API `/api/devices` endpoint. The IoT Center server app, in turn, sends a request to the InfluxDB `/api/v2/authorizations` endpoint to retrieve authorizations with the prefix `IoT Center: `.

{{/* Source: /app/ui/src/pages/DevicesPage.ts */}}

```js
// Source: github/iot-center-v2/app/server/influxdb/authorizations.js

/**
 * Gets all authorizations.
 * @return promise with an array of authorizations
 * @see https://influxdata.github.io/influxdb-client-js/influxdb-client-apis.authorization.html
 * @return {Array<import('@influxdata/influxdb-client-apis').Authorization>}
 */
async function getAuthorizations() {
  const {id: orgID} = await getOrganization()
  const authorizations = await authorizationsAPI.getAuthorizations({orgID})
  return authorizations.authorizations || []
}

/**
 * Gets all IoT Center device authorizations.
 * @return promise with an authorization or undefined
 * @see https://influxdata.github.io/influxdb-client-js/influxdb-client-apis.authorization.html
 * @return {Array<import('@influxdata/influxdb-client-apis').Authorization>}
 */
async function getIoTAuthorizations() {
  const authorizations = await getAuthorizations()
  const {id: bucketId} = await getBucket()
  return authorizations.filter(
    (val) =>
      val.description.startsWith(DESC_PREFIX) &&
      isBucketRWAuthorized(val, bucketId)
  )
}
```

### IoT Center: register a device

  - Click "Register" button.
  - UI calls `/api/devices/DEVICE_ID`. Calls `/api/v2/authorizations` to find a matching authorization.
  - If the device is not registered, IoT Dev Center API sends a request to the
    InfluxDB API `/api/v2/authorizations` endpoint to create an API token with the name `IoT Center: DEVICE_ID`. IoT Dev Center app uses the `DESC_PREFIX` (`= "IoT Center: "`) constant variable to identify and retrieve registered devices. The IoT Center app will be responsible for expiring, rotating, and managing tokens. The IoT device will use the token to authenticate API requests to InfluxDB.

    #### Create an authorization

    ```js
    const {AuthorizationsAPI} = require('@influxdata/influxdb-client-apis')
    const {getOrganization} = require('./organizations')
    const {getBucket} = require('./buckets')
    const authorizationsAPI = new AuthorizationsAPI(influxdb)
    const DESC_PREFIX = 'IoT Center: '
    const CREATE_BUCKET_SPECIFIC_AUTHORIZATIONS = false

    /**
     * Creates authorization for a supplied deviceId
     * @param {string} deviceId client identifier
     * @return {import('@influxdata/influxdb-client-apis').Authorization} promise with authorization or an error
     */
    async function createIoTAuthorization(deviceId) {
      const {id: orgID} = await getOrganization()
      let bucketID = undefined
      if (CREATE_BUCKET_SPECIFIC_AUTHORIZATIONS) {
        bucketID = await getBucket(INFLUX_BUCKET).id
      }
      console.log(
        `createIoTAuthorization: deviceId=${deviceId} orgID=${orgID} bucketID=${bucketID}`
      )
      return await authorizationsAPI.postAuthorizations({
        body: {
          orgID,
          description: DESC_PREFIX + deviceId,
          permissions: [
            {
              action: 'read',
              resource: {type: 'buckets', id: bucketID, orgID},
            },
            {
              action: 'write',
              resource: {type: 'buckets', id: bucketID, orgID},
            },
          ],
        },
      })
    }
        ```
### IoT Center: demo the virtual device

  - The IoT Center **virtual device** emulates a real IoT device by generating measurement data and writing the data to InfluxDB. Use the virtual device to demonstrate the IoT Center dashboard and the InfluxDB API before you advance to adding physical devices or other clients.

### IoT Center: remove a device
  - Sends an HTTP `DELETE` request to the IoT Dev Center `/api/devices/<deviceID>` endpoint.
  - Device settings
    - Device configuration details are composed of your InfluxDB configuration and authorization details for the Device ID.

## Write data to InfluxDB

See [Write data with the API](/influxdb/v2.1/write-data/developer-tools/api/)

### IoT Center: write IoT data

  IoT Center provides a "Write Missing Data" button that generates `environment` (temperature, humidity, pressure, CO2, TVOC, latitude, and longitude) [measurement]() data for the virtual device. The button generates measurements for every minute over the last seven days and writes the generated measurement data to the InfluxDB bucket you configured.

  To write the measurements to the InfluxDB bucket, the IoT Center UI calls [`getWriteApi(org, bucket, precision, options)`]() to instantiate a point writer from the [influxdb-client-js]() Javascript client library.

  [`writeApi.writePoint(point)`]() adds each new point to an array in a `WriteBuffer` object.

  [`writeApi.flush()`]() batches the points (to optimize network performance) and passes each batch in an HTTP POST request to the InfluxDB `/api/v2/write` endpoint.

  ```js
  if (totalPoints > 0) {
    const batchSize = 2000
    const url =
      write_endpoint && write_endpoint !== '/mqtt' ? write_endpoint : '/influx'
    const influxDB = new InfluxDB({url, token})
    const writeApi = influxDB.getWriteApi(org, bucket, 'ns', {
      batchSize: batchSize + 1,
      defaultTags: {clientId: id},
    })
    try {
      // write random temperatures
      const point = new Point('environment') // reuse the same point to spare memory
      onProgress(0, 0, totalPoints)

      const writePoint = async (time: number) => {
        const gpx = getGPX(time)
        point
          .floatField('Temperature', generateTemperature(time))
          .floatField('Humidity', generateHumidity(time))
          .floatField('Pressure', generatePressure(time))
          .intField('CO2', generateCO2(time))
          .intField('TVOC', generateTVOC(time))
          .floatField('Lat', gpx[0] || state.config.default_lat || 50.0873254)
          .floatField('Lon', gpx[1] || state.config.default_lon || 14.4071543)
          .tag('TemperatureSensor', 'virtual_TemperatureSensor')
          .tag('HumiditySensor', 'virtual_HumiditySensor')
          .tag('PressureSensor', 'virtual_PressureSensor')
          .tag('CO2Sensor', 'virtual_CO2Sensor')
          .tag('TVOCSensor', 'virtual_TVOCSensor')
          .tag('GPSSensor', 'virtual_GPSSensor')
          .timestamp(String(time) + '000000')
        writeApi.writePoint(point)

        pointsWritten++
        if (pointsWritten % batchSize === 0) {
          await writeApi.flush()
          onProgress(
            (pointsWritten / totalPoints) * 100,
            pointsWritten,
            totalPoints
          )
        }
      }

      if (missingDataTimeStamps && missingDataTimeStamps.length)
        for (const timestamp of missingDataTimeStamps)
          await writePoint(timestamp)
      else
        while (lastTime < toTime) {
          lastTime += 60_000 // emulate next minute
          await writePoint(lastTime)
        }
      await writeApi.flush()
    } finally {
      await writeApi.close()
    }
    onProgress(100, pointsWritten, totalPoints)
  }
  ```

  ## Advanced topics
    1. Segment your data

## Query InfluxDB

See [Query with the API](/influxdb/v2.1/query-data/execute-queries/influx-api/)

## Query data for visualizations

### Query with Flux

   ```js
   import "influxdata/influxdb/v1"    
   from(bucket: "iot_center")
     |> range(start: ${fluxDuration(timeStart)})
     |> filter(fn: (r) => r._measurement == "environment")
     |> filter(fn: (r) => r["_field"] == "Temperature" or r["_field"] == "TVOC" or r["_field"] == "Pressure" or r["_field"] == "Humidity" or r["_field"] == "CO2")
     |> filter(fn: (r) => r.clientId == "virtual_device")
     |> v1.fieldsAsCols()
    ```

#### Query result table
  | _start | _stop | _time | CO2Sensor | GPSSensor | HumiditySensor | PressureSensor | TVOCSensor | TemperatureSensor | _measurement | clientId | CO2 | Humidity | Pressure | TVOC | Temperature |
  |:----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|


### IoT device dashboard

IoT Center provides a dashboard of data visualizations for each registered device.
To view the device dashboard, from the "Virtual Device" page, click the "Device Dashboard" button.
To generate the dashboard, the IoT Center UI queries device measurements in
the InfluxDB bucket.

IoT Center uses the `queryTable` data structure create visualizations.
To instantiate a query API configuration, the IoT Center `DashboardPage` calls  [`InfluxDB({url: '/influx', token}).getQueryApi(org)`]() from the [influxdb-client-js]() Javascript client library.

`queryTable(queryApi, query, options)` passes a [Flux]() query in an HTTP POST request to the InfluxDB `api/v2/query` endpoint and returns query results.

```ts
// Source: iot-center-v2/app/ui/src/pages/DashboardPage.tsx

const fetchDeviceMeasurements = async (
 config: DeviceConfig,
 timeStart = '-30d'
): Promise<GiraffeTable> => {
 const {
   // influx_url: url, // use '/influx' proxy to avoid problem with InfluxDB v2 Beta (Docker)
   influx_token: token,
   influx_org: org,
   influx_bucket: bucket,
   id,
 } = config
 const queryApi = new InfluxDB({url: '/influx', token}).getQueryApi(org)
 const result = await queryTable(
   queryApi,
   flux`
 import "influxdata/influxdb/v1"    
 from(bucket: ${bucket})
   |> range(start: ${fluxDuration(timeStart)})
   |> filter(fn: (r) => r._measurement == "environment")
   |> filter(fn: (r) => r["_field"] == "Temperature" or r["_field"] == "TVOC" or r["_field"] == "Pressure" or r["_field"] == "Humidity" or r["_field"] == "CO2")
   |> filter(fn: (r) => r.clientId == ${id})
   |> v1.fieldsAsCols()`
 )
 return result
}
```

The `GiraffeTable` (alias `Table`) interface:

```ts
// Source: iot-center-v2/app/node_modules/@influxdata/giraffe/src/types/index.ts

export interface Table {
  getColumn: GetColumn
  getColumnName: (columnKey: string) => string | null // null if the column is not available
  getColumnType: (columnKey: string) => ColumnType | null // null if the column is not available
  getOriginalColumnType: (columnKey: string) => FluxDataType | null // null if the column is not available
  columnKeys: string[]
  length: number
  addColumn: (
    columnKey: string,
    fluxDataType: FluxDataType,
    type: ColumnType,
    data: ColumnData,
    name?: string
  ) => Table
}
```

   ## Advanced topics
     1. Optimize your queries
1. Manage and secure tokens
   1. InfluxDB Cloud (token access restrictions)
2. Monitor the IoT app
2. Tasks
2. Dashboards
