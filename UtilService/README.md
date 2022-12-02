# Utility Service
This template concerns the installation of the utilities server which can be used for several tasks for a microservice.
## Overview
This template consists of the following part(s):
> ### Service definer
|SERVER||
|:-|-|
|`USoft.Utilities`|Which is the server who can be used to run several jobs / tasks for a microservice.|
## Installation
Install the template by downloading the wizard and changing the names.\
All the steps will be explained here.\
This is also where you can change the names of the fields.\
After the wizard is done new files will be generated ending with _replaced.xml.
1. Import the 1_server_USOFT_UTILITIES_replaced.xml file in the service definer under Tools > SQL Command with SQL statement:
`invoke xml.import with select <path>\1_server_USOFT_UTILITIES_replaced.xml`
The newly imported template will be ready for use in the service definer.
## How to use
1. Install the template
2. In the service definer: Install the USoft.Utilities server in the service definer under Actions > Install
3. Publish and restart the USoft.Utilities server under Publish > Publish and Restart
## Change log
> #### Created by [Peter Ekkelenkamp](mailto:peter.ekkelenkamp@usoft.com)
|Version|Author|Date|Description|
|:---|:---|:---|:---|
|v1.0|[Peter Ekkelenkamp](mailto:peter.ekkelenkamp@usoft.com) |`2022-11-11`|`Initial commit`|