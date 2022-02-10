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
   2. Emulate IoT devices
      - In IoT Dev Center, you can register __virtual devices__ that emulate real IoT devices and demonstrate the IoT Dev Center app before you advance to physical devices or other clients.
1. Register a virtual IoT device
    ## Authorization and authentication in InfluxDB
     IoT devices generate measurement data....
     To write time series measurements to InfluxDB, your application or device needs to be authorized.
     An InfluxDB **authorization** consists of a set of permissions and an API token.
     To authenticate InfluxDB API requests, the device passes the API token in the `Authorization` request header.
   - On "Device Registrations" load, ...
     - IoT Dev Center UI sends a request to the IoT Dev Center API `/api/env/<deviceID>` endpoint to retrieve `IoT Center: ` authorizations. IoT Dev Center server If the device is registered, i.e. it has an authorization in InfluxDB, then the IoT Center API returns authorization and InfluxDB configuration properties as configuration details for the device.

     #### Get authorizations

          ```js
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
   - On "Register", ...
     - If the device is not registered, IoT Dev Center API sends a request to the
        InfluxDB API `/api/v2/authorizations` endpoint to create an API token with the name `IoT Center: DEVICE_ID`. IoT Dev Center app uses the `DESC_PREFIX` (`= "IoT Center: "`) constant variable to identify and retrieve registered devices. The new token will be used in future API requests and token management (e.g. for expiration and rotation).

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
   - Remove a device
     - Sends an HTTP `DELETE` request to the IoT Dev Center `/api/devices/<deviceID>` endpoint.
   - Device settings
     - Device configuration details are composed of your InfluxDB configuration and authorization details for the Device ID.

1. Write IoT data
  IoT Dev Center provides a "Write Missing Data" button that generates virtual IoT device `environment` [measurements]() for temperature, humidity, pressure, CO2, TVOC, latitude, and longitude for every minute over the last seven days. It writes the generated measurement data to your configured InfluxDB bucket.

  ## Write data to InfluxDB


  #### nodejs

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

2. Device dashboard
   IoT Center provides a "Device Dashboard" button for each registered device.
   To generate the dashboard, the IoT Center UI queries device measurements in
   the InfluxDB bucket.

   ## Query data in InfluxDB


   #### nodejs

     To instantiate a query API configuration, the IoT Center `DashboardPage` calls  [`InfluxDB({url: '/influx', token}).getQueryApi(org)`]() from the [influxdb-client-js]() Javascript client library.

     `queryTable(queryApi, query, options)` passes a [Flux]() query in an HTTP POST request to the InfluxDB `api/v2/query` endpoint and returns query results.

     ```js
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
   ## Advanced topics
     1. Optimize your queries
1. Manage and secure tokens
   1. InfluxDB Cloud (token access restrictions)
2. Monitor the IoT app
2. Tasks
2. Dashboards
