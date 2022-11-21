# Manufacturing Ontologies

## Introduction

An ontology defines the language used to describe a system. In the manufacturing domain, these systems can represent a factory or plant but also enterprise applications or supply chains. There are several established ontologies in the manufacturing domain. Most of them have long been standardized. In this repository, we have focused on two of these ontologies, namely ISA95 to describe a factory ontology and IEC 63278 Asset Administration Shell to describe a manufacturing supply chain. Furthermore, we have included a factory simulation and an end-to-end solution architecture for you to try out the ontologies, leveraging IEC 62541 OPC UA and the Microsoft Azure Cloud.

## Digital Twin Definition Language

The ontologies defined in this repository are described by leveraging the Digital Twin Definition Language (DTDL), which is specified [here](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md).

## International Society of Automation 95 (ISA95)

The ISA95 standard is described [here](https://en.wikipedia.org/wiki/ANSI/ISA-95).

## IEC 63278 Asset Administration Shell (AAS)

The IEC 63278 Asset Administration Shell is described [here](https://www.plattform-i40.de/IP/Redaktion/EN/Standardartikel/specification-administrationshell.html).

## IEC 62541 Open Platform Communications Unified Architecture (OPC UA)

IEC 62541 Open Platform Communications Unified Architecture (OPC UA) is described [here](https://opcfoundation.org). 

## Overall Solution Architecture

<img src="Docs/architecture.png" alt="architecture" width="900" />

## Production Line Simulation

This repository also contains a production line simulation made up of several Stations, leveraging the machine information model described above, as well as a simple Manufacturing Execution System (MES). Both the Stations and the MES are containerized for easy deployment.

### UA Cloud Twin

The simulation makes use of the UA Cloud Twin also available from the Digital Twin Consortium [here](https://github.com/digitaltwinconsortium/UA-CloudTwin). It automatically detects OPC UA assets from the OPC UA telemetry messages sent to the cloud and registers ISA95-compatible digital twins in Azure Digital Twins service for you.

<img src="Docs/twingraph.png" alt="twingraph" width="900" />

#### Mapping OPC UA Servers to the ISA95 Hierarchy Model

UA Cloud Twin takes the combination of the OPC UA Application URI and the OPC UA Namespace URIs discovered in the OPC UA telemetry stream and creates ISA95 Work Center assets for each one.

#### Mapping OPC UA PubSub Publishers to the ISA95 Hierarchy Model

UA Cloud Twin takes the OPC UA Publisher ID and creates ISA95 Area assets for each one.

#### Mapping OPC UA PubSub Datasets to the ISA95 Hierarchy Model

UA Cloud Twin takes each OPC UA Field discovered in the received Dataset metadata and creates an ISA95 Work Unit asset for each.

### Default Simulation Configuration

The simulation is configured to include 8 production lines. The default configuration is depicted below:

| Production Line | Ideal Cycle Time (in seconds) |
|:---------------:|:-----------------------------:|
| Munich | 6 |
| Capetown | 8 |
| Mumbai | 11 |
| Seattle |	6 |
| Beijing 1	| 9 |
| Beijing 2	| 8 |
| Beijing 3	| 4 |
| Rio |	10 |

### OPC UA Node IDs of Station OPC UA Server

The following OPC UA Node IDs are used in the Station OPC UA Server for telemetry to the cloud
* i=379 - manufactured product serial number
* i=385 - number of manufactured products
* i=391 - number of discarded products
* i=398 - running time
* i=399 - faulty time
* i=400 - status (0=station ready to do work, 1=work in progress, 2=work done and good part manufactured, 3=work done and scrap manufactured, 4=station in fault state)
* i=406 - energy consumption
* i=412 - ideal cycle time
* i=418 - actual cycle time
* i=434 - pressure

### Automatic Installation of Production Line Simulation and Cloud Services

Simply click on the button below, it will deploy all required resources (on Microsoft Azure):

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fdigitaltwinconsortium%2FManufacturingOntologies%2Fmain%2FDeployment%2Farm.json)

Once the deployment is complete in the Azure Portal, please follow these steps to configure the production line simulation:

1. Login to the deployed VM using the credentials you provided during deployment and download and install Docker Desktop from [here](https://www.docker.com/products/docker-desktop), including the Windows Subsystem for Linux (WSL) integration. After installation and a required system restart, accept the license terms and install the WSL2 Linux kernel by following the instructions. Then verify that Docker Desktop is running in the Windows System Tray and enable Kubernetes in Settings.

2. On the VM, browse to [here](https://github.com/digitaltwinconsortium/ManufacturingOntologies) and select Code -> Download Zip. Unzip the contents to a directory of your choice. Navigate to the OnPremAssets directory of the Zip you just downloaded and run the StartSimulation.cmd script from the OnPremAssets folder in a cmd prompt by supplying the primary key connection string of your Event Hubs namespace and the Azure region you picked during deployment as parameters. The primary key connection string can be read in the Azure Portal under your Event Hubs' "share access policy" -> "RootManagedSharedAccessKey".

3. Under Access Control -> Role Assignments of your Azure Digital Twin service instance in the Azure Portal, add a new Role Assignment of type "Azure Digital Twins Data Owner", assign it's access to "Managed Identity" and under "Select Users", select your previously deployed Azure Web App service instance.

Please note: If you update your Docker Desktop runtime environment, you will need to stop and restart the simulation!

### Next Steps

If you want to add a 3D viewer to the simulation, you can follow the steps to configure the 3D Scenes Studio outlined [here](https://learn.microsoft.com/en-us/azure/digital-twins/how-to-use-3d-scenes-studio) and map the 3D robot model from [here](https://cardboardresources.blob.core.windows.net/public/RobotArms.glb) to the digital twins automatically generated by the UA Cloud Twin:

<img src="Docs/3dviewer.png" alt="3dviewer" width="900" />

If you want to calculate OEE, add no-code dashboards or make predictions about the production, set up the [Data History](https://learn.microsoft.com/en-us/azure/digital-twins/concepts-data-history) feature in the Azure Digital Twins service to historize your contextualized OPC UA data to Azure Data Explorer deployed in this solution. You can find the wizard to set this up in the Azure Digital Twins service configuration in the Azure portal. 

If you want to test a "digital feedback loop", i.e. triggering a command on one of the OPC UA servers in the simulation from the cloud, based on a time-series reaching a certain threshold (the simulated pressure), then configure and run the StartUACloudCommander.bat file and deploy the PressureRelief Azure Function in your Azure subscription and create an application registration for your ADX instance as described [here](https://docs.microsoft.com/en-us/azure/data-explorer/provision-azure-ad-app). You also need to define the following environment variables in the Azure portal for the Function:

* ADX_INSTANCE_URL
* ADX_DB_NAME
* AAD_TENANT_ID
* APPLICATION_KEY
* APPLICATION_ID
* IOT_HUB_NAME
* IOT_HUB_KEY
* UACOMMANDER_NAME
* UA_SERVER_ENDPOINT
* UA_SERVER_METHOD_ID
* UA_SERVER_OBJECT_ID
* UA_SERVER_DNS_NAME

### Replacing the Production Line Simulation with a Real Production Line

Once you are ready to connect your own production line, simply delete the VM through the Azure Portal or, if you are running the simulation on a local PC, call the StopSimulation.cmd script. Then run UA Cloud Publisher on a Docker-enabled edge gateway PC (on Windows, for Linux, remove the "c:" bits) with the following command. The PC needs Internet access (via port 8883) and needs to be able to connect to your OPC UA-enabled machiens in your production line:

    docker run -itd -v c:/publisher/logs:/app/logs -v c:/publisher/settings:/app/settings -p 80:80 ghcr.io/barnstee/ua-cloudpublisher:main

In this case, UA Cloud Publisher stores its configuration and log files locally on the Edge PC under c:/publisher on Windows or /publisher on Linux.

Then, open a browser on the Edge PC and navigate to http://localhost. You are now connected to the UA Cloud Publisher's interactive UI. Select the Configuration menu item and enter the following information, replacing [myeventhubsnamespace] with the name of your Event Hubs namespace and replacing [myeventhubsnamespaceprimarykeyconnectionstring] with the primary key connection string of your Event Hubs namespace. The primary key connection string can be read in the Azure Portal under your Event Hubs' "share access policy" -> "RootManagedSharedAccessKey". Then click Update:
  
    BrokerClientName: UACloudPublisher  
    BrokerUrl: [myeventhubsnamespace].servicebus.windows.net
    BrokerPort: 9093  
    BrokerUsername: $ConnectionString  
    BrokerPassword: [myeventhubsnamespaceprimarykeyconnectionstring]  
    BrokerMessageTopic: data  
    BrokerMetadataTopic: metadata  
    SendUAMetadata: true  
    MetadataSendInterval: 30  
    BrokerCommandTopic: commands  
    BrokerResponseTopic: response  
    BrokerMessageSize: 262144  
    CreateBrokerSASToken: false  
    UseTLS: false  
    PublisherName: UACloudPublisher  
    InternalQueueCapacity: 1000  
    DefaultSendIntervalSeconds: 1  
    DiagnosticsLoggingInterval: 30  
    DefaultOpcSamplingInterval: 500  
    DefaultOpcPublishingInterval: 1000  
    UAStackTraceMask: 645  
    ReversiblePubSubEncoding: false  
    AutoLoadPersistedNodes: true  

Next, we will configure the OPC UA data nodes from your machines (or connectivity adapter software). To do so, select the OPC UA Server Connect menu item, enter the OPC UA server IP address and port and click Connect. You can now browse the OPC UA Server you want to send telemetry data from. If you have found the OPC UA node you want, right click it and select publish.

That's it! You can check what is currently being published by selecting the Publishes Nodes menu item. You can also see diagnostics information from UA Cloud Publisher on the Diagnostics menu item.

## License

<a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" /></a>

This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>.
