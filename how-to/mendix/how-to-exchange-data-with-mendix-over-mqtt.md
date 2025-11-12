# How-To: Exchange data with Mendix over MQTT
## Setup an MQTT connection between Mendix and Intelligence Hub for communicating over a message broker.
### What Does This Article Cover?

This article describes one option for exchanging data between HighByte Intelligence Hub and the Mendix low-code application platform. Multiple integration approaches exist between Intelligence Hub and Mendix, depending on whether data flows from devices to Mendix or from Mendix to Intelligence Hub. This article focuses on communicating over MQTT between Mendix and Intelligence Hub. It assumes the reader has familiarity working with both.

### What is Mendix?

Mendix is a low-code platform designed to rapidly build, deploy, and manage enterprise-grade web and mobile applications. Applications built in Mendix can integrate with external systems and data sources through many connectors and adapters like MQTT, making it a strong fit for connecting industrial data pipelines developed in HighByte Intelligence Hub.

### Intelligence Hub MQTT capabilities

HighByte Intelligence Hub has various connectors including an MQTT client. Separate and adjacent to that, there is an MQTT broker included with each runtime. The MQTT client can connect to any MQTT server but for this article, the included Intelligence Hub MQTT server is used for convenience. Here we go over connecting Mendix MQTT client and Intelligence Hub MQTT client to this broker for the two to exchange data.

### Configuring Intelligence Hub MQTT server

Refer to user guide: https://guide.highbyte.com/setup/application_settings/#mqtt-broker

<img width="1158" height="706" alt="image" src="https://github.com/user-attachments/assets/c7bfb67f-04f4-4757-85bc-0cf93c500406" />

### Configuring Intelligence Hub MQTT client

Refer to user guide: https://guide.highbyte.com/configuration/connect/connections/mqtt/

And starter project to import the following pipeline into your runtime: https://support.highbyte.com/kb/getting-started/case-study/tutorial-starter-solution

<img width="1678" height="621" alt="image" src="https://github.com/user-attachments/assets/2619502c-7e02-4d0d-90d9-e5f321a9d086" />

The starter project will publish to the following topic path which can be subscribed to from the Mendix side:

<img width="1050" height="425" alt="image" src="https://github.com/user-attachments/assets/2bf0403a-82f3-459b-b77f-1ccba5ab91af" />

### Configuring Mendix MQTT Integration
- Download the Mendix MQTT Connector from the Mendix Marketplace to your Mendix app from https://marketplace.mendix.com/link/component/119508 
- Follow the set-up instructions for the MQTT Connector from https://docs.mendix.com/appstore/modules/mqtt/ 
- Create a microflow to subscribe to the MQTT Topic that you set up in HighByte 
- Retrieve the MqttConnector.ConnectionDetail object that matches your HighByte configuration 
- Use the ‘Subscribe to MQTT’ action 
- Fill in the parameters and specify the Topic 
- Set the ‘On message microflow’ parameter to the microflow that will be triggered for every new MQTT message 
- In the ‘New message microflow’, add two input parameters called ‘Payload’ and ‘Topic’, both as String matching those exact names starting with uppercase 

> [!NOTE]
> The parameters must be called ‘Payload’ and ‘Topic’

- The Payload will contain a JSON with the MQTT message 
- Either save this directly, or use an Import Mapping 

### Next steps

You should now be able to communicate over MQTT between Mendix and the Intelligence Hub. Please feel free to reach out with any questions by submitting a ticket in your portal. You may next want to explore how this connection can be useful with the applications you build in Mendix. Some ideas could be to publish the latest machine states from Intelligence Hub to dashboard the current state in Mendix. Check out our reference articles on examples. You may also want to explore the other modes of communication depending on the data access needs. 

### Additional Resources
- [Mendix MQTT Connector](https://marketplace.mendix.com/link/component/119508%C2%A0)
- [Mendix MQTT Connector setup](https://docs.mendix.com/appstore/modules/mqtt/%C2%A0)
- [Mendix Docs – Import Mappings](https://docs.mendix.com/refguide/import-mappings/)
- [Mendix Academy – How to create an Import Mapping](https://academy.mendix.com/link/modules/138/lectures/1234/5.3.2-Create-Import-Mapping)
- [Intelligence Hub MQTT broker](https://guide.highbyte.com/setup/application_settings/#mqtt-broker)
- [Intelligence Hub MQTT client](https://guide.highbyte.com/configuration/connect/connections/mqtt/)
- [Tutorial: Starter Solution](https://support.highbyte.com/kb/getting-started/case-study/tutorial-starter-solution)
