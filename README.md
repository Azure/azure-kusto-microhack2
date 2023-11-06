# Microhack 2: Data Exploration and Visualization with KQL

This Microhack is organized into the following 3 challenges:

- Challenge 4: Explore and transform data
- Challenge 5: Advanced KQL operators
- Challenge 6: Visualisation

Each challenge has a set of tasks that need to be completed in order to move on to the next challenge. It is advisable to complete the challenges and tasks in the prescribed order.

---

## In order to receive the ADX microhack digital badge, you will need to complete the challenges marked with 🎓. Please submit the KQL queries/commands of these challenges in the following link: [Answer sheet - ADX Microhack 2](https://forms.office.com/r/RkyRWgVN0G)

---

## Challenge 4: Explore and transform data

### Expected Learning Outcomes

- Create an Update Policy to transform the data at ingestion time

For the next tasks, we will use the `LogisticsTelemetry` table (which obtains data from the Event Hub).

---

### Task 1: Create an update policy 🎓

By taking 10 records, we can see that the telemetry column has a JSON structure. In this task, we will use an 'Update Policy' to manipulate the raw data in the `LogisticsTelemetry` table (the source table) and transform the JSON data into separate columns, that will be ingested into a new table that we’ll create (referred to as the "target" or "destination" table).

#### What is an Update Policy?

Update Policy is like an internal Extract-Transform-Load (ETL) step. It can help you manipulate or enrich the data as it gets ingested into the source table (e.g. extracting JSON into separate columns, creating a new calculated column, joining the new records with a static dimension table that is already in your database, etc). For these cases, using an update policy is a very common and powerful practice.

Each time records get ingested into the source table, the update policy's query (which we'll define in the Update Policy) will run on these records (and only on newly ingested records - other existing records in the source table aren't visible to the Update Policy when it runs). The results of the query will be appended to the target table.

Though ADX is purpose-built for interactive, ad-hoc queries over nested JSON data, parsing the JSON data into typed columns at ingestion time can improve performance and reduce query complexity for end users - particularly if you have a lot of queries that need to access the same fields.

We want to create a new table (called `LogisticsTelemetryManipulated`), with a calculated column (we will call it: `NumOfTagsCalculated`) that contains the result of the calculation: `telemetry.TotalTags - telemetry.LostTags`

The schema of the new (destination) table will be:

```Kusto
(deviceId:string, enqueuedTime:datetime, NumOfTagsCalculated:int, Temp:real)
```

![Demo Destination Schema](/assets/images/Challenge4-Task1-Pic1.png)

As a reminder, our telemetry data looks like this (note that the order of the keys may be different):

```JSON
{
  "BatteryLife": 73,
  "Light": "70720.236143472212",
  "Tilt": "18.608539012789223",
  "Humidity": "60.178854625386215",
  "Shock": "-4.6141182265359628",
  "Pressure": "529.61165751122712",
  "ActiveTags": 165,
  "TransportationMode": "Ocean",
  "Status": "Online",
  "LostTags": 9,
  "Temp": "7.5054504554744765",
  "TotalTags": 185,
  "Location": {
    "alt": 1361.0469,
    "lon": -107.7473,
    "lat": 36.0845
  }
}
```

#### Step 1: Build the target table

✍🏻 Run this command to create the target table.

```Kusto
.create table LogisticsTelemetryManipulated (deviceId:string, enqueuedTime:datetime, NumOfTagsCalculated:long, Temp:real)
```

#### Step 2: Create a Function for the Update Policy 🎓

✍🏻 Complete the function that extracts the desired columns and types.

🕵🏻 Hint: You can test the query outside of the `.create-or-alter function` function to verify it works as expected. Then paste the logic into the function to create it.

```Kusto
.create-or-alter function ManipulateLogisticsTelemetryData()
{
    <Complete the query>
}
```

#### Step 3: Create the Update Policy 🎓

✍🏻 Now that the transformation function has been created, let's make a change to the update policy of the destination table to invoke our new function for each new batch of data.

🕵🏻 Hint: Refer to the docs for the syntax of the update policy.

```Kusto
<Complete the command>
```

#### Step 4: Make sure the data is transformed correctly in the destination table

```Kusto
LogisticsTelemetryManipulated
| take 10
```

At the end of this activity, you should see the data from the source table `LogisticsTelemetry` transformed into the destination table `LogisticsTelemetryManipulated`.

References:

- [Kusto update policy](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/updatepolicy)

---

---

## Challenge 5: Going more advanced with KQL

### Task 1: Declaring variables 🎓

📆 Use table: `LogisticsTelemetryHistorical`

✍🏻 Use a **'let'** statement to create a list of the 10 device ids which have the highest Shock. Then, use this list in a following query to find the total average temperature of these 10 devices.

You can use the **'let'** statement to set a variable name equal to an expression or a function.

let statements are useful for:

- Breaking up a complex expression into multiple parts, each represented by a variable.
- Defining constants outside of the query body for readability.
- Defining a variable once and using it multiple times within a query.

🕵🏻 Hint 1: [in operator](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/in-cs-operator#subquery)

🕵🏻 Hint 2: [let](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/letstatement#examples)

🕵🏻 Hint 3: Remember to include a semicolon (`;`) at the end of your let statement.

---

### Task 2: Add more fields to your time chart 🎓

📆 Use table: `LogisticsTelemetryHistorical`

✍🏻 Write a query to show a time chart of the historical number of records, by transportation mode. Use 10 minute bins.

Example result:

![Transport Mode Time Chart](/assets/images/chart-4.png)

---

### Task 3: Some Geo-Mapping

📆 Use table: `LogisticsTelemetryHistorical`

✍🏻 Write a query to show on a map, the locations (based on the longitude and latitude) of 10 devices with the highest temperature in the last 90 days.

🕵🏻 Hint 1: `top` operator

🕵🏻 Hint 2: `render scatterchart with (kind=map)`

Once the map is displayed, you can click on the locations. Note that in order to show more details in the balloon, you need to change the render phrase to include `series=<TempColumn>`.

Example result:

![Geo-Mapping](/assets/images/Challenge5-Task3-map.png)

References:

- [render operator with scatter chart](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/renderoperator?pivots=azuredataexplorer)

---

### Task 4: Range

Range is a tabular operator; it generates a single-column table of values, whose values are start, start + step, ... up to and until stop.

Run the following query and review the results:
`range MyNumbers from 1 to 8 step 2`

Range also works with dates:
`range LastWeek from ago(7d) to now() step 1d`

We will use the range operator as part of the time series creation in the next tasks.

#### Machine Learning with Kusto and Time Series Analysis

Many interesting use-cases use machine learning algorithms and derive interesting insights from telemetry data. Often, these algorithms require a strictly structured dataset as their input. The raw log data usually doesn't match the required structure and size. We will see how we can use the `make-series` operator to create well curated time-series data.

Then, we can use built in functions like [series_decompose_anomalies](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/series-decompose-anomaliesfunction). Anomalies (outliers) will be detected by the Kusto service and highlighted as red dots on the time series chart.

#### What is a Time Series?

A time series is a collection of observations of well-defined data items obtained through repeated measurements over time and listed in time order. Most commonly, the data points are consistently measured at equally spaced intervals. For example, measuring the temperature of the room each minute of the day would comprise a time series. Data collected irregularly is not a time series.

#### What is Time Series Analysis?

Time series analysis comprises of methods for analyzing time series data in order to extract meaningful statistics and other characteristics of the data. Time series forecasting, for example, is the use of a model to predict future values based on previously observed values.

#### What is Time Series Decomposition?

Time series decomposition involves thinking of a series as a combination of 4 components:

1. **trends** (increasing or decreasing value in the series)

2. **seasonality** (repeating short-term cycle in the series)

3. **baseline** (the predicted value of the series, which is the sum of seasonal and trend components)

4. **noise** (The residual random variation in the series).

We can use built-in functions, that uses time series decomposition to forecast future metric values and/or detect anomalous values.

This is what a time series looks like:

![What a time series looks like](/assets/images/Challenge5-Task4-Pic1.png)

#### Why should you use series instead of the summarize operator?

The summarize operator does not add "null bins" - rows for time bin values for which there's no corresponding row in the table. It's a good idea to "pad" the table with those bins. Advanced built-in ML capabilities like anomaly detection need the data points to be consistently measured at equally spaced intervals. The **make-series** can create such a "complete" series.

---

### Task 5: Anomaly Detection 🎓

📆 Use table: `LogisticsTelemetryHistorical`

✍🏻 Write a query to create an anomaly chart of the average shock.

For this task, we will provide more instructions. To generate these series, start with:

```Kusto
let min_t = (toscalar(LogisticsTelemetryHistorical | summarize min(enqueuedTime)));
let max_t = (toscalar(LogisticsTelemetryHistorical | summarize max(enqueuedTime)));
let step_interval = 10m;
LogisticsTelemetryHistorical
| make-series avg_shock_series=avg(Shock) on (enqueuedTime) from (min_t) to (max_t) step step_interval
```

Now, we will use this `avg_shock_series` and run `series_decompose_anomalies`.
This built-in function takes an expression containing a series (_dynamic_ numerical array) as input, and extracts anomalous points with scores.

```Kusto
| extend anomalies_flags = series_decompose_anomalies(avg_shock_series, 1)
| render anomalychart with(anomalycolumns=anomalies_flags, title='Avg Shock Anomalies')
```

The anomalies/outliers can be clearly spotted in the 'anomalies_flags' points.

Example result:

![Anomaly Detection on Average Shock](/assets/images/Challenge5-Task4-anomalies.png)

🤔 Wait! It looks a lot like it simply figured out the highest and lowest Shock values. Can you tweak the parameters to make it less sensitive?

References:

- [make-series](https://docs.microsoft.com/en-us/azure/data-explorer/time-series-analysis)
- [ADX Anomaly Detection](https://docs.microsoft.com/en-us/azure/data-explorer/anomaly-detection#time-series-anomaly-detection)

---

💡 **FOR THE NEXT TASKS, WE WILL USE THE NYC TAXI DATA**

The NYC Taxi data was ingested into the `NYCTaxiRides` table during Microhack 1. If the proctor hasn't provided the data set, use this Azure Open Dataset on [NYC Taxi Rides](https://docs.microsoft.com/en-us/azure/open-datasets/dataset-taxi-yellow?tabs=azureml-opendatasets) by ingesting it into your ADX cluster.

---

### Task 6: Get familiar with the new table and create a pie chart 🎓

📆 Use table: `NYCTaxiRides`

Write some queries to get familiar with this table.

✍🏻 After some familiarity, write a query to create a pie chart of the payments type. Use `tostring()` to convert the `payment_type` to a string before rendering the pie chart.

For now, the payment types will be represented by numbers. We will join the payment type table in a later task.

Example result:

![Payment Type Pie Chart](/assets/images/taxi-pie.png)

---

### Task 7: Datetime operations 🎓

📆 Use table: `NYCTaxiRides`

✍🏻 Write a query to create a column chart which will show the number of rides for each day of the week, across the entire data set. You can use 1, 2, ..., 7 to denote Sunday through Saturday.

Example result:

![Datetime Operations](/assets/images/taxi-days.png)

References:

- [dayofweek()](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/dayofweekfunction)
- [datetime_part()](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/datetime-partfunction)

---

### Task 8: Multiple series on the same time chart

📆 Use table: `NYCTaxiRides`

✍🏻 Write a query to find out if the tip amount correlates with the number of passengers in the taxi between 1 July 2022 and 31 July 2022. Restrict the number of passengers to a maximum of 4.

🕵🏻 Hint: How is average tip amount changing against passenger count and pickup time?

Example result:

![Multiple Series Same Time chart](/assets/images/taxi_passengers.png)

---

### Task 9: Detect anomalies in the tip amount 🎓

📆 Use table: `NYCTaxiRides`

✍🏻 Write a query to draw an anomaly chart for the tip amount in the month of July 2022.

🕵🏻 Hint 1: `make-series` for the average tip amount, with 1-hour steps

🕵🏻 Hint 2: Use `series_decompose_anomalies` with this series and parameter of `5` (sensitivity level)

Example result:

![Tip Amount Anomalies](/assets/images/tip_anomaly.png)

---

### Task 10: External Data 🎓

📆 Use table: `NYCTaxiRides`

The `externaldata` operator returns a table whose schema is defined in the query itself, and whose data is directly read from an external storage artifact, such as a blob in Azure Blob Storage, a file in Azure Data Lake Storage, or even a file in a GitHub repository. Since the data is not being ingested into ADX, it cannot be indexed, compressed, or stored in the hot cache. For best performance, we recommend that data be ingested. External data can, however, be used in sporadic cases, where you do not want to ingest the data.

Take a look at this csv file: [https://raw.githubusercontent.com/Azure/azure-kusto-microhack/main/assets/ExternalData/payment_type_lookup.csv](https://raw.githubusercontent.com/Azure/azure-kusto-microhack/main/assets/ExternalData/payment_type_lookup.csv).

The file represents the mapping between the numeric code of the payment type and its description.

✍🏻 Here is how we can use KQL to handle this external data:

```Kusto
let payment_type_lookup_data = (externaldata (code:string,description:string)
[ @"https://raw.githubusercontent.com/Azure/azure-kusto-microhack/main/assets/ExternalData/payment_type_lookup.csv" ]
with(format="csv", ignoreFirstRecord=true))
| project tolong(code), description;
payment_type_lookup_data
```

References:

- [externaldata operator](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/externaldata-operator?pivots=azuredataexplorer)

---

### Task 11: Let's **join** the party 🎓

📆 Use table: `NYCTaxiRides`

The taxi rides table has a field of `Payment_type`. This is a numeric code signifying how the passenger paid for the trip. Use the `payment_type_lookup`, to join between the payment code and the description. Use a _leftouter join_ to merge the rows of the two tables to form a new table, by matching values of the payment code column.

✍🏻 Peform the join operation and render a time chart of the number of records, per payment type over time, with 1-day bins, based on data between 2022-07-01 and 2022-07-31.

🤔 What is the most common method of payment for rides? Credit cards or cash? What does it look like over time?

Example result:

![Payment Types Joined in Time chart](/assets/images/Challenge5-Task9-Pic1.png)

References:

- [join operator](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/joinoperator?pivots=azuredataexplorer)

---

### Task 12: Forecasting

📆 Use table: `NYCTaxiRides`

This task will comprise of multiple steps that lead to one overall query.

✍🏻 Create a time chart that will show:

- The number of rides during July 2022
- A forecast of the number of drive-ins for the first week of August, based on July 2022 (use the `series_decompose_forecast` function).
- Once a series is created, you can render a time chart.

Additional Information:

- 🕵🏻 Hint 1: Start your query with:

  ```Kusto
  let min_t = datetime(2022-07-01);
  let max_t = datetime(2022-08-07); // Note that there is no data in the first week of August. We will forecast the data for this week.
  NYCTaxiRides
  | where tpep_dropoff_datetime between (min_t .. max_t)
  ```

- 🕵🏻 Hint 2: Make a series of the number of rides, on `tpep_pickup_datetime` between these dates. Use steps of 30 minutes.

- 🕵🏻 Hint 3: Use `series_decompose_forecast` with parameters of this series and second parameter of: `24*7`. The second parameter is an Integer specifying the number of points at the end of the series to predict (forecast). These points are excluded from the learning (regression) process. We will use `24*7` additional data points, in order to forecast a week forward.

Example result:

![Forecasting](/assets/images/forecast.png)

---

---

## Challenge 6: Visualisation

### Task 1: Prepare interactive dashboards with ADX Dashboards 🎓

✍🏻 Using the Dashboard feature of Azure Data Explorer, build a dashboard using outputs of any 5 queries (on the `LogisticsTelemetryHistorical` table) that you have created in the previous challenges with the following improvements:

- Add a filter on the dashboard so that the user can choose the timespan
- Add a filter on the dashboard so that the user can choose the transportation mode

Include **filters for the dashboard** so that the queries do not need to be modified if the user wants to analyze the charts with different values of a parameter. For example, users would like to analyze the charts over the last week, the last 14 days as well as the last 1 month. Users would also like to analyze the charts by different transportation modes.

🕵🏻 Hint 1: In the query window, explore the "Share" menu.

![Query Share Option](/assets/images/Challenge6-Task1-Pic1.png)

Example dashboards:

![Example Dashboard 1](/assets/images/Challenge6-Task1-dashboard.png)

![Example Dashboard 2](/assets/images/Challenge6-Task1-dashboard2.png)

References:

- [Visualize data with the Azure Data Explorer dashboard](https://docs.microsoft.com/en-us/azure/data-explorer/azure-data-explorer-dashboards)
- [Parameters in Azure Data Explorer dashboards](https://docs.microsoft.com/en-us/azure/data-explorer/dashboard-parameters)

---

### Task 2: Prepare Management Dashboard with Power BI

✍🏻 Visualize the outputs of [Task 5 (Pie Chart)](#task-6-get-familiar-with-the-new-table-and-create-a-piechart) and [Task 7 (Datetime Operations)](#task-7-datetime-operations) in Challenge 5 in Power BI using the **DirectQuery** mode.

🕵🏻 Hint 1: In the query window, explore the "Share" menu.

There are multiple ways to connect ADX and Power BI depending on the use case. For this microhack, we will use the DirectQuery method. Feel free to explore other methods listed in the docs.

References:

- [Visualize data using the Azure Data Explorer connector for Power BI](https://docs.microsoft.com/en-us/azure/data-explorer/power-bi-connector)
- [Visualize data using a query imported into Power BI](https://docs.microsoft.com/en-us/azure/data-explorer/power-bi-imported-query)

---

🎉 Congrats! You've completed the second Microhack. Keep going with [Microhack 3: Advanced KQL, Policies, Security](https://github.com/Azure/azure-kusto-microhack3)
