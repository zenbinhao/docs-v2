---
title: Basic modeling and filtering
list_title: Basic modeling
description: >
  A use case of modeling data from a reviews system.
menu:
  influxdb_2_0:
    name: Basic modeling
    parent: Query with Flux
weight: 209
related:
---
<!-- list_query_example: percentages -->

## Use case: online store reviews

The company Amazing has an online shop that allows users to buy and review products.
A review can be submitted using different *channels*: iOS, Android, and Web.
To provide feedback the users evaluate their satisfaction by rating the product with **stars**, an integer value from 1 to 5.
In addition, the system automatically verifies if the user bought the product or not.

Each time a user provides a review, the Amazing system collects the following information:

- `product_id`: the indentifier of the product reviewed;
- `user_id`: the identifier of the user that submitted the review;
- `stars`: the satisfaction index of the user;
- `channel`: the channel through with the review is submitted;
- `verified`: indicates whether purchase of the product was verified by the system; and
- `timestamp`: Unix timestamp associated with the review.

The following table contains some example reviews:

| `user_id` | `product_id` | `channel` | `verified` | `stars` | `timestamp`         |
|-----------|--------------|-----------|------------|---------|---------------------|
| 4         | 3            | iOS       | true       | 4       | 1577834352191000000 |
| 2         | 1            | Android   | true       | 5       | 1577834714745000000 |
| 4         | 3            | iOS       | false      | 4       | 1577920448087000000 |
| 1         | 2            | iOS       | false      | 5       | 1577921483448000000 |
| 1         | 2            | Android   | false      | 2       | 1577922010760000000 |

## Goal

The goal of this Flux example is to extract basic information about the product reviews from this data.
For instance, we might want to select *only verified reviews*,
or all reviews below 3 stars.

## Before you begin

Know how to run InfluxDB.
See [Get started]().

<!-- - An account on [InfluxDB Cloud](https://eu-central-1-1.aws.cloud2.influxdata.com/login) -->
<!-- - Familiarity with the concept of time series -->

## Steps

Use the data provided in [`data/ratings.txt`](https://github.com/influxdata/docs-v2/blob/master/static/downloads/influxdb-k8-minikube.yaml).

1. Prepare the data
2. Store the data in InfluxDB
3. Select a time interval of the data
4. Filter the data

### Prepare the data
#### Simple Data Modeling

Model the data so we can use it withInfluxDB.

See [doc write data](https://v2.docs.influxdata.com/v2.0/write-data/).

<!-- TODO link to line protocol

An InfluxDB database stores data points.
A data point has four components:

- measurement: description and namespace for the point.
- tagset: key/value string pairs usually used to identify the point.
- fieldset: key/value pairs that are typed and usually contain the point data.
- timestamp: date and time associated with the fields.
 -->

Identify the measurement, tagset, fieldset, and timestamp for our use case.

**##### Solution [TO BE HIDDEN FROM THE READER]**
- measurement: "ratings"
- tagset: user_id, product_id, channel, verified
- fieldset: stars
- timestamp: date

#### InfluxDB data point serialization

The serialization format for data points is defined by the line protocol.
An example of data point from the specification helps to explain the terminology:

```
measurement,tag1=tag1Value,tag2=tag2Value field1=field1Value,field2=field2Value timestamp
```

The timestamp has to be in UTC and Unix format (up to nanosecond precision).

<!-- **##### Let's cook** -->
Serialize the reviews reported in the table above using line protocol.

<!-- **##### Solution [TO BE HIDDEN FROM THE READER]** -->

```
ratings,user_id=4,product_id=3,channel=iOS,verified=true stars=4 1577834352191000000

ratings,user_id=2,product_id=1,channel=Android,verified=true stars=5 1577834714745000000
ratings,user_id=4,product_id=3,channel=iOS,verified=false stars=4 1577920448087000000
ratings,user_id=1,product_id=2,channel=iOS,verified=false stars=5 1577921483448000000
ratings,user_id=1,product_id=2,channel=Android,verified=false stars=2 1577922010760000000
```

### Store the data in InfluxDB

Load data into InfluxDB.
<!-- InfluxDB is provided with Flux, a composable, easy to learn, and highly productive data scripting language. -->
All that we have to do is to **log-in** into the InfluxDB web interface, select the **data** section, and create a new bucket called **ratings**.
Then, the option **add data** allows us to load the data directly in line protocol format.

<!-- 
**##### Note**
- Start Time: 2020-01-01T00:00:00Z
- Stop Time: 2020-01-20T00:00:00Z
-->

### Select a time interval of the data

The first element to specify in a query is the bucket from which to take the data.
This is what the [`from()`]() function is for.
In our case, we use the **ratings** bucket.

```
from(bucket: "ratings")
```

Then, since all the reviews loaded before have temporal dependencies,
we need to select the time range of data to show using the [`range`]() function.
There are two options to define a range:

**Absolute**: reviews produced between January 1, 2020 and January 20, 2020

```
from(bucket: "ratings")
    |> range(start:2020-01-01T00:00:00Z, stop: 2020-01-04T00:00:00Z)
```

**Relative**: reviews produced in the last 2 hours.

```
from(bucket: "ratings")
    |> range(start:-2h)
```

The pipe forward operator `|>` is used to concatenate all the steps of the script.

<!-- **##### Let's cook** -->

Select all the reviews produced between January 1, 2020 and January 20, 2020.

<!-- **##### Solution [TO BE HIDDEN FROM THE READER]** -->

```
from(bucket: "ratings")
    |> range(start:2020-01-01T00:00:00Z, stop: 2020-01-05T00:00:00Z)
```

### Filter the data

#### Filter by Tag

In the following example, we use Flux to select the reviews
provided through the Android channel
between January 1, 2020 and January 20, 2020.

```
from(bucket: "ratings")
    |> range(start:2020-01-01T00:00:00Z, stop: 2020-01-20T00:00:00Z)
    |> filter(fn: (r) => r._measurement == "ratings")
    |> filter(fn: (r) => r._field == "stars")
    |> filter(fn: (r) => r.channel == "Android")
```

The [`filter()`]() function filters data based on conditions defined in a predicate function `(fn)`.
It takes in input `(r)` that represents a single table row and it outputs if the current row should be selected or not.
This script selects only the rows with `channel = Android`.
Filtering on the `measurement` and `_field` fields is a best practice since a bucket can contain multiple measurements or fields.

<!-- **##### Let's cook** -->

Write a query to select only the verified reviews between January 1, 2020 and January 20, 2020.

<!-- **##### Solution [TO BE HIDDEN FROM THE READER]** -->

```
from(bucket: "ratings")
    |> range(start:2020-01-01T00:00:00Z, stop: 2020-01-20T00:00:00Z)
    |> filter(fn: (r) => r._measurement == "ratings")
    |> filter(fn: (r) => r._field == "stars")
    |> filter(fn: (r) => r.verified == "true")
```

#### Filter by Value

In the following example, John wants to select the reviews provided through the Android channel with value greater or equal to 3 between January 1, 2020 and January 20, 2020.

```
from(bucket: "ratings")
    |> range(start:2020-01-01T00:00:00Z, stop: 2020-01-20T00:00:00Z)
    |> filter(fn: (r) => r._measurement == "ratings")
    |> filter(fn: (r) => r._field == "stars")
    |> filter(fn: (r) => r.channel == "Android")
    |> filter(fn: (r) => r._value >= 3)
```

To access the value of stars, we have to use the **_value** field.

<!-- **##### Let's cook** -->

Write a query to select only the poor reviews—less than 3 stars—between January 1, 2020 and January 20, 2020.

<!-- ****##### Solution [TO BE HIDDEN FROM THE READER]** -->

```
from(bucket: "ratings")
    |> range(start:2020-01-01T00:00:00Z, stop: 2020-01-20T00:00:00Z)
    |> filter(fn: (r) => r._measurement == "ratings")
    |> filter(fn: (r) => r._field == "stars")
    |> filter(fn: (r) => r._value < 3)
```

<!--
Based on work by:
Alessio Bernardo - alessio.bernardo@polimi.it
Emanuele Falzone - emanuele.falzone@polimi.it
Andrea Mauri - andrea.mauri@quantiaconsulting.com
-->

