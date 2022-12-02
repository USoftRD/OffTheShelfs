# Application Config Setting
> ##### This template concerns the installation of a publication server.
> ##### After this installation, the user is able to design and publish web pages in the web designer.
## Overview
> ##### This template consists of the following parts:
### Service definer
> Publication server
> > ##### Which is the server who publish the in the web designer developed webpages.
### Webdesigner
> Publication configuration
> > ##### To publish the webpages in the webdesigner, a configuration is required to deploy the webpages in the same structure as defined in the publication server.
## Installation
> ##### Install the template by downloading the wizard and changing the names.
> ##### All the steps will be explained here.
> ##### This is also where you can change the names of the fields.
> ##### After the wizard is done new files will be generated ending with _replaced.xml.
> > 1. Import the 1_server_USOFT_PUBLICATIONS_replaced.xml file in the service definer under Tools > SQL Command with SQL statement:
> > > `invoke xml.import with select '<path>\1_server_USOFT_PUBLICATIONS_replaced.xml`

> > 2. Check the PUBLIC class
> > 3. Import the 1_publicationconfiguration_USOFT_replaced.xml file in the webdesigner under Tools > SQL Command with SQL statement:
> > >`invoke xml.import with select '<path>\1_publicationconfiguration_USOFT_replaced.xml`

> ##### The newly imported template will be ready for use in the definer and the webdesigner.
## How to use
> > 1. Install the template
> > 2. In the service definer:
	- Install the Publications server in the service definer under Actions > Install
	- Publish and restart the Publications server under Publish > Publish and Restart
> > 3. Publish in the webdesigner the publication configuration under Publish > Publish... with Application checked and the correct Publication Confguration choosen
> > 4. With an internet browser, test the following URL:
> > > - https://localhost:8081/PUBLIC/USOFT_APP_RS/app.usoft
## Created by
> ##### [Danny Dorstijn](mailto:danny.dorstijn@usoft.com)
## Change log
|Version|Date|Author|description|
|  ---  |--- | ---  | --- |
|v1.0|2022-10-28|[Danny Dorstijn](mailto:danny.dorstijn@usoft.com)|Initial commit|