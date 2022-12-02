# Application Config Setting
This template concerns the objects to use a table for aplication settings, which can be used to get values for variables used in SQL-statements or so.
## Overview
This template consists of the following part(s):
> ### Definer
|TABLE||
|-|-|
|`APPLICATION_CONFIG_SETTING`|Holds the names and values for each config setting|
|`USER_CONFIG_SETTING`|Holds the names, values and value type for each config setting, which can be used to publish to users (by webdesigner, for example)|

|COMPONENT||
|-|-|
|`USER_CONFIG_SETTING_VALIDATION`| component which can validate an user setting by the format the value type is set by regular expression, like numbers, date, etc.|

|CONSTRAINT||
|-|-|
|`USER_CONFIG_SETTING_VALIDATION`|A constraint which execute the user config setting validation component to validate the value input for table user config setting.|
## Installation
Install the template by downloading the wizard and changing the names.\
All the steps will be explained here.\
This is also where you can change the names of the fields.\
After the wizard is done new files will be generated ending with _replaced.xml.\

1. Import the **output\1_table_ApplicationConfigSetting_replaced.xml** file in the definer under Teamwork > Object Shopping > Import from file...
2. Import the **output\2_component_UserConfigSettingValidation_replaced.xml** file in the definer under Teamwork > Object Shopping > Import from file...
3. Import the **output\3_constraint_UserConfigSettingValidation_replaced.xml** file in the definer under Teamwork > Object Shopping > Import from file...

The newly imported template will be ready for use in the definer.
## How to use
1. Install the template in definer
2. 'Create tables...' in binder
3. Open Authorizer and run 'Fill Authorizer Tables...'
4. In application, open 'Objects...' to explore defined tables, pick 'Application Config Settings'
5. Enter for 1 or more configuration setting(s):|- Name (required)
* Value (required)
* Explanation
* User Setting y/n
6. In application, open 'Objects...' to explore defined tables, pick 'User Config Settings'
7. Enter for 1 or more configuration setting(s):
* Name (required)
* Value (required)
* Explanation
* User Setting y/n
* Data type
* Max/min string length
* Regular Expression
* Validation failed message
* Notes

These 'Application Config| Settings' or 'User Config Settings' records can, for example, be used as settings in SQL-statements\
Example: \
> `select acs.value from APPLICATION_CONFIG_SETTING acs where acs.name = '<acs_name>'`
## Change log
> #### Created by [Peter Ekkelenkamp](mailto:peter.ekkelenkamp@usoft.com)
|Version|Author|Date|Description|
|:---|:---|:---|:---|
|v1.0|[Peter Ekkelenkamp](mailto:peter.ekkelenkamp@usoft.com) |`2022-10-28`|`Initial commit`|