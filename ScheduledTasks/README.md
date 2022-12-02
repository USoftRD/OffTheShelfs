# Application Config Setting
> ##### This template concerns the installation of the scheduled tasks functionality where a server, on a set time, will run one or more defined jobs.
## Overview
> ##### This template consists of the following parts:
### Definer
> SCHEDULED_TASK table
> > ##### Holds the schedules tasks defined with jobs which will run by the server.
> SCHEDULED_RUN job
> > ##### A job which will run by the server, which will run other defined jobs.
### Service definer
> USoft.Utilities server
> > ##### Which is the server who will run and start the scheduled run job, which which will run other defined jobs on the defined times in the scheduled tasks table.
## Installation
> ##### Install the template by downloading the wizard and changing the names.
> ##### All the steps will be explained here.
> ##### This is also where you can change the names of the fields.
> ##### After the wizard is done new files will be generated ending with _replaced.xml.
> > 1. Import the 1_table_SCHEDULED_TASK_replaced.xml file in the definer under Teamwork > Import from file
> > 2. Import the 2_job_SCHEDULE_RUN_replaced.xml file in the definer under Teamwork > Import from file
> > 3. Validate SCHEDULE_RUN job in the definer
> > 4. Import the 1_server_USOFT_UTILITIES_replaced.xml file in the service definer under Tools > SQL Command with SQL statement:
> > > `invoke xml.import with select '<path>\1_server_USOFT_UTILITIES_replaced.xml`

> > 5. Check the classes and used SQL statements in the USoft.Utilities server
> ##### The newly imported template will be ready for use in the definer and the service definer.
## How to use
> > 1. Install the template
> > 2. In the service definer:
> > > - Install the USoft.Utilities server in the service definer under Actions > Install
> > > - Publish and restart the USoft.Utilities server under Publish > Publish and Restart

> > 3. In the application enter an scheduled task record to run a job on a scheduled time per day
> > 4. Check if the defined job have run on the defined scheduled time in the scheduled tasks table
## Created by
> ##### [Peter Ekkelenkamp](mailto:peter.ekkelenkamp@usoft.com)
## Change log
|Version||Date||Author||Description|
|:-|:-|:-|:-|:-|:-|:-|
|v1.0|	|2022-11-11|	|[Peter Ekkelenkamp](mailto:peter.ekkelenkamp@usoft.com) |	|Initial commit|