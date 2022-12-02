# JSON API Service
||
|-|
|This template concerns the installation of a JSON API server for a microservice to publish data in a JSON format.|
|This default service consists data from 2 tables (main and sub table).|
## Overview
||
|-|
|This template consists of the following part(s):|
> ### Definer
||||
|-|-|-|
|TABLE|||
||`OUT_JSON`|Holds a data set of 4 columns, which can be redefined in definer to fine tune.|
||`OUT_JSON_SUB`|Which is a child table of OUT_JSON table and holds a data set of 3 columns, which can be redefined in definer to fine tune|
> ### Service definer
||||
|-|-|-|
|SERVER|||
||`JsonAPI`|Which is the server who publish the above data set from tables parent OUT_JSON and child OUT_JSON_SUB|
|||It holds an example to generate the JSON output, as leading data from tables parent OUT_JSON and child OUT_JSON_SUB|
|||After redefine the tables in definer (like: more or less columns) it is possible and required to rescript the JSON class for a correct JSON output|
## Installation
||
|-|
|Install the template by downloading the wizard and changing the names.|
|All the steps will be explained here.|
|This is also where you can change the names of the fields.|
|After the wizard is done new files will be generated ending with _replaced.xml.|
||
|-|
|1. Import the 1_table_OUT_JSON_replaced.xml file in the definer under Teamwork > Import from file...|
|2. Import the 1_table_OUT_JSON_SUB_replaced.xml file in the definer under Teamwork > Import from file...|
|3. Import the 1_server_JsonAPI_replaced.xml file in the service definer under Tools > SQL Command with SQL statement:|
|`invoke xml.import with select '<path>\1_server_JsonAPI_replaced.xml`|
|4. Check the jsonClass class and `USOFT_GET_OUT_JSON, USOFT_GET_OUT_JSON_BY_ID, USOFT_GET_OUT_JSON_SUB_BY_ID` SQL statements|

> The newly imported template will be ready for use in the definer and the service definer.
## How to use
|||
|-|-|
|1. Install the template||
|2. In the service definer:|Install the JsonAPI server in the service definer under Actions > Install|
||Publish and restart the JsonAPI server under Publish > Publish and Restart|
|3. In the application:|Add some data in the OUT_JSON and OUT_JSON_SUB tables|
|4. With Postman, test the following URLs:| http://localhost:8091/api/json/outjsons |
|| http://localhost:8091/api/json/outjson/{id} |
|| http://localhost:8091/api/json/outjsonsub/{id} |
|| http://localhost:8091/api/json/outjsonssubs |
## Change log
> #### Created by [Peter Ekkelenkamp](mailto:peter.ekkelenkamp@usoft.com)
|Version|Author|Date|Description|
|:---|:---|:---|:---|
|v1.0|[Peter Ekkelenkamp](mailto:peter.ekkelenkamp@usoft.com) |`2022-11-04`|`Initial commit`|