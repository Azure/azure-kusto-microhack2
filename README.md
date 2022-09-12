## Microhack 2: Data exploration and visualization with KQL (Preview)

This Microhack is organized into the following 3 challenges:
- Challenge 5: Explore and transform data
- Challenge 6: Advanced KQL operators
- Challenge 7: Visualisation

Each challenge has a set of tasks that need to be completed in order to move on to the next challenge. It is advisable to complete the challenges and tasks in the prescribed order.

---
In order to receive the ADX microhack digital badge, you will need to complete the challenges marked with 🎓. Please submit the KQL queries/commands of these challenges in the following link: [Answer sheet - ADX Microhack 2](https://forms.office.com/r/RkyRWgVN0G)
---

---
### Challenge 5: Explore and transform data
  
**Expected Learning Outcomes:**
- Create an update policy to transform the data at ingestion time
  
For the next task, we will use the LogisticsTelemetry table (which obtains data from the Event Hub).

---
##### Task 1: Create an update policy 🎓
  
By taking 10 records, we can see that the telemetry column has a JSON structure. In this task, we will use an 'update policy' to manipulate the raw data in the LogisticsTelemetry table (the source table) and transform the JSON data into separate columns, that will be ingested into a new table that we’ll create (“target table”).</br>
Update policy is like an internal ETL. It can help you manipulate or enrich the data as it gets ingested into the source table (e.g. extracting JSON into separate columns, creating a new calculated column, joining the new records with a static dimension table that is already in your database, etc). For these cases, using an update policy is a very common and powerful practice. </br>
Each time records get ingested into the source table, the update policy's qeury (which we'll define in the update policy) will run on them (and only on newly ingested records - other existing records in the source table aren’t visible to the update policy when it runs), and the results of the query will be appended to the target table. </br>
We want to create a new table, with a calculated column (we will call it: NumOfTagsCalculated) that contains the following value: telemetry.TotalTags - telemetry.LostTags. </br>

The schema of the new (destination) table would be:
```
  ( deviceId:string, enqueuedTime:datetime, NumOfTagsCalculated:int, Temp:real)
```

  
  ![Screen capture 1](/assets/images/Challenge3-Task2-Pic1.png)
  
  Example (note that the order of the keys may be different):
  
  ```
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
  
  **Build the target table**
  
``` 
.create table LogisticsTelemetryManipulated  (deviceId:string, enqueuedTime:datetime, NumOfTagsCalculated:long, Temp:real) 
```
  
  **Create a function for the update policy** 🎓
  
``` 
.create-or-alter function ManipulateLogisticsTelemetryData()
{ 
     <Complete the query>
} 
```
    
  **Create the update policy** 🎓
``` 
     <Complete the command>
```

  **Make sure the data is transformed correctly in the destination table**
 ``` 
    LogisticsTelemetryManipulated
    | take 10
```
  
**Relevant docs for this challenge:**
  - [Kusto update policy - Azure Data Explorer | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/updatepolicy)

---
---
### Challenge 6: Going more advanced with KQL

#### Task 1: Declaring variables 🎓
Use a **'let'** statement to create a list of the 10 device Ids which have the highest Shock. Then, use this list in a following query to find the total average temperature of these 10 devices.

You can use the **'let'** statement to set a variable name equal to an expression or a function.
let statements are useful for:
- Breaking up a complex expression into multiple parts, each represented by a variable.
- Defining constants outside of the query body for readability.
- Defining a variable once and using it multiple times within a query.

Hint 1: [in operator - Azure Data Explorer | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/in-cs-operator#subquery)

Hint 2: [let - Azure Data Explorer | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/letstatement#examples)

Hint 3: Remember to include a ";" at the end of your let statement.

---
#### Task 2: Add more fields to your timechart 🎓
Write a query to show a timechart of the number of records, by TransportationMode. Use 10 minute bins.

Expected result:
 ![Screen capture 1](/assets/images/chart-4.png)


---
#### Task 3: Some geo-mapping
Write a query to show on map the locations (based on the longitude and latitude) of 10 devices with the highest temperature from the last 90 days.
<br>
Hint 1: 'top' operator </br>
Hint 2: render scatterchart with (kind = map)

Once the map is displayed, you can click on the locations. Note that in order to show more details in the balloon, you need to change the render phrase to include 'series=<TempColumn>'.

[render operator with scatter chart](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/renderoperator?pivots=azuredataexplorer)
   
  Expected result:
  
<img src="/assets/images/Challenge6-Task3-map.png" width="400">

---
#### Task 4: Range 
Range is a tabular operator: it generates a single-column table of values, whose values are start, start + step, ... up to and until stop.
Run the following query and review the results:

range MyNumbers from 1 to 8 step 2

Range also works with dates:
range LastWeek from ago(7d) to now() step 1d

We will use the range operator as part of the time series creation in the next tasks.
  
---
#### Machine learning with Kusto and time series analysis

Many interesting use cases use machine learning algorithms and derive interesting insights from telemetry data. Often, these algorithms require a strictly structured dataset as their input. The raw log data usually doesn't match the required structure and size. We will see how we can use the make-series operator to create well curated data (time series).

Then, we can use built in functions like [series_decompose_anomalies](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/series-decompose-anomaliesfunction). Anomalies/ outliers will be detected by the Kusto service and highlighted as red dots on the time series chart.

**Time series:**
What is a time series?
A time series is a collection of observations of well-defined data items obtained through repeated measurements over time and listed in time order. Most commonly, the data points are consistently measured at equally spaced intervals. For example, measuring the temperature of the room each minute of the day would comprise a time series. Data collected irregularly is not a time series.

**What is time series analysis?**

Time series analysis comprises methods for analyzing time series data in order to extract meaningful statistics and other characteristics of the data. Time series forecasting, for example, is the use of a model to predict future values based on previously observed values. 

**What is time series decomposition?**
Time series decomposition involves thinking of a series as a combination of 4 components: 
- trends (increasing or decreasing value in the series)
- seasonality (repeating short-term cycle in the series)
- baseline (the predicted value of the series, which is the sum of seasonal and trend components) 
- noise (The residual random variation in the series). 
We can use built in functions, that uses time series decomposition to forecast future metric values and/or detect anomalous values.

This is what time series looks like:

![Screen capture 1](/assets/images/Challenge6-Task4-Pic1.png)

**Why should you use series instead of the summarize operator?**

The summarize operator does not add "null bins" — rows for time bin values for which there's no corresponding row in the table. It's a good idea to "pad" the table with those bins. Advanced built in ML capabilities like anomaly detection need the data points to be consistently measured at equally spaced intervals. The **make-series** can create such a “complete” series.

---
#### Task 5: Anomaly detection 🎓
Write a query to create an anomaly chart of the average shock.

For this task, we will provide more instructions:

To generate these series, start with:
```
let min_t = (toscalar(LogisticsTelemetryHistorical | summarize min(enqueuedTime)));
let max_t = (toscalar(LogisticsTelemetryHistorical | summarize max(enqueuedTime)));
let step_interval = 10m;
LogisticsTelemetryHistorical
| make-series avg_shock_series=avg(Shock) on (enqueuedTime) from (min_t) to (max_t) step step_interval 
```
Now, we will use this avg_shock_series and run series_decompose_anomalies.
This built-in function takes an expression containing a series (dynamic numerical array) as input, and extracts anomalous points with scores.
```
| extend anomalies_flags = series_decompose_anomalies(avg_shock_series, 1) 
| render anomalychart  with(anomalycolumns=anomalies_flags, title='avg shock anomalies') 
```
The anomalies/outliers can be clearly spotted in the 'anomalies_flags' points.

[make-series](https://docs.microsoft.com/en-us/azure/data-explorer/time-series-analysis) <br>
[ADX Anomaly Detection](https://docs.microsoft.com/en-us/azure/data-explorer/anomaly-detection#time-series-anomaly-detection)
  
Expected result:
  
<img src="/assets/images/Challenge6-Task4-anomalies.png" width="650">
</br></br>

💡 **FOR THE NEXT TASKS, WE WILL USE the NYC TAXI DATA.**  <br>

If the proctor hasn't provided the data set, use this Azure Open Dataset on [NYC Taxi Rides](https://docs.microsoft.com/en-us/azure/open-datasets/dataset-taxi-yellow?tabs=azureml-opendatasets) to ingest this data into your ADX cluster.

---
#### Task 6: Get familiar with the new table and create a piechart 🎓
Write some queries to get familiar with this table. After some familiarity, write a query to create a piechart of the payments type. Use 'tostring' to convert the payment_type to string before rendering the piechart.

Expected result:</br>
<img src="/assets/images/taxi-pie.png" width="500">

---
#### Task 7: Datetime operations 🎓
Write a query to create a columnchart which will show the number of rides for each day of the week, across the entire data set.  You can use 1, 2, ..., 7 to denote Sunday through Saturday.

[dayofweek() - Azure Data Explorer | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/dayofweekfunction)
[datetime_part() - Azure Data Explorer | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/datetime-partfunction)
  
Expected result:</br>
<img src="/assets/images/taxi-days.png" width="650">

---
#### Task 8: Multiple series on the same timechart
Write a query to find out if the tip amount correlates with the number of passengers in the taxi between 1 July 2021 and 31 July 2021. Restrict the number of passengers to maximum of 4.

Hint: How is average tip amount changing against passenger count and pickup time 

Expected result:</br>
<img src="/assets/images/taxi_passengers.png" width="650">

---
#### Task 9: Detect anomalies in the tip amount 🎓
Write a query to draw an anomaly chart for the tip amount in the month of July 2021. <br>
Hint 1: make-series for the average tip amount, with 1 h steps <br>
Hint 2: Use series_decompose_anomalies with this series and parameter of 5 (sensitivity level)

Expected result:</br>
<img src="/assets/images/tip_anomaly.png" width="650">

---
#### Task 10: External data 🎓

The externaldata operator returns a table whose schema is defined in the query itself, and whose data is directly read from an external storage artifact, such as a blob in Azure Blob Storage, a file in Azure Data Lake Storage, or even a file in GitHub repository. Since the data is not being ingested into ADX, it cannot be indexed, compressed, or stored in the hot cache. For best performance, we recommend that data be ingested. External data can, however, be used in sporadic cases, where you do not want to ingest the data.</br>
Take a look at this csv file: https://raw.githubusercontent.com/Azure/azure-kusto-microhack/main/assets/ExternalData/payment_type_lookup.csv.
The file represents the mapping between the numeric code of the payment type and its description </br>
Here is how we can use KQL to handle this external data:

```
let payment_type_lookup_data = (externaldata (code:string,description:string)
[ @"https://raw.githubusercontent.com/Azure/azure-kusto-microhack/main/assets/ExternalData/payment_type_lookup.csv" ]
with(format="csv", ignoreFirstRecord=true))
| project tolong(code), description;
payment_type_lookup_data
```
  
[externaldata operator - Azure Data Explorer | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/externaldata-operator?pivots=azuredataexplorer)
  
---
#### Task 11: Let's **join** the party 🎓
The taxi rides table has a field of Payment_type. This is a numeric code signifying how the passenger paid for the trip. Use the payment_type_lookup, to join between the 
payment code and the description. Use a leftouter join to merge the rows of the two tables to form a new table, by matching values of the payment code column.

Render a time chart of the number of records, per payment type over time, with 1 day bins, based on data between 2021-07-01 and 2021-07-31. 
What is the most common method of payment for rides? Credit cards or cash? What does it look like over time? 

[join operator - Azure Data Explorer | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/joinoperator?pivots=azuredataexplorer)

Expected result:
  
 <img src="/assets/images/Challenge6-Task9-Pic1.png" width="650">
  
---
#### Task 12: Forecasting
Create a timechart that will show:
- The number of rides during July 2021
- A forecast of the number of drive-ins for the first week of August, based on July 2021 (Use the series_decompose_forecast function).

Hint: Start your query with:

  ```
let min_t = datetime(2021-07-01);
let max_t = datetime(2021-08-07); // Note that there is no data in the first week of August. We will forecast the data for this week.
taxi
| where tpep_dropoff_datetime between (min_t .. max_t) 
```
  
- Make a series of the number of rides, on tpep_pickup_datetime between these dates. Use steps of 30 minutes. 
- Use series_decompose_forecast with parameters of this series and second parameter of: '24\*7' (The second parameter is an Integer specifying the number of points at the end of the series to predict (forecast). These points are excluded from the learning (regression) process. We will use '24\*7` additional data points, in order to forecast a week forward).
- Once a series is created, you can render a timechart.

Expected result: </br>
<img src="/assets/images/forecast.png" width="650">

---
---
### Challenge 7: Visualisation

---
#### Task 1: Prepare interactive dashboards with ADX Dashboard 🎓

Using the Dashboard feature of Azure Data Explorer, build a dashboard using outputs of any 5 queries (on LogisticsTelemetryHistorical table) that you have created in the previous challenges with the following improvements:
  - Add filter on the dashboard so that the user can choose the timespan
  - Add filter on the dashboard so that the user can choose the transportation mode

Include **filters for the dashboard** so that the queries do not need to be modified if the user wants to analyze the charts with different values of a parameter. For example, users would like to analyze the charts over the last week, the last 14 days as well as the last 1 month. Users would also like to analyze the charts by different transportation modes.

Hint 1: In the query window, explore the “Share” menu.
  
  ![Screen capture 1](/assets/images/Challenge7-Task1-Pic1.png)
 
- [Visualize data with the Azure Data Explorer dashboard | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-explorer/azure-data-explorer-dashboards)
- [Parameters in Azure Data Explorer dashboards | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-explorer/dashboard-parameters)

<img src="/assets/images/Challenge7-Task1-dashboard.png" width="500">
<img src="/assets/images/Challenge7-Task1-dashboard2.png" width="500">
  
---
#### Task 2: Prepare management dashboard with PowerBI
Visualize the outputs of [Task 5](https://github.com/Azure/azure-kusto-microhack#task-5-get-familiar-with-the-new-table-and-create-a-piechart) and [Task 6](https://github.com/Azure/azure-kusto-microhack#task-6-datetime-operations) in Challenge 6 in PowerBI using the DirectQuery mode. 

Hint 1: In the query window, explore the “Share” menu.

There are multiple ways to connect ADX and PowerBI depending on the use case. For this Microhack, we will use the DirectQuery method. Feel free to explore other methods on the docs.

[Visualize data using the Azure Data Explorer connector for Power BI | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-explorer/power-bi-connector)

[Visualize data using a query imported into Power BI | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-explorer/power-bi-imported-query)

---
🎉 Congrats! You've completed the second Microhack. Keep going with [Microhack 3: Advanced KQL, Policies, Security](https://github.com/Azure/azure-kusto-microhack3)

