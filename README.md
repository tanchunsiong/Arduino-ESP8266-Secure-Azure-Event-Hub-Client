#Securely stream data from ESP8266 MCUs to Azure IoT Hub over HTTPS/REST

###References
This project is a fork of [Dave Grove's project](https://github.com/gloveboxes/Arduino-ESP8266-Secure-Azure-IoT-Hub-Client), and has been slightly modified to make use of Event Hub and Different Types of Sensors.

###Purpose

This solution securely streams sensor data directly to 
[Azure Event Hub](https://azure.microsoft.com/en-us/services/event-hubs/) over HTTPS calling Azure REST APIs from ESP8266 MCUs.


###Device Platform


The [ESP8266](https://en.wikipedia.org/wiki/ESP8266) is a device based out of Espressif ESP8266 which comes with integrated WiFi. 



###Firmware

[Arduino core for ESP8266 WiFi chip V2.0](https://github.com/esp8266/Arduino) firmware adds HTTPS ([TLS](http://axtls.sourceforge.net/) 1.0 and 1.1) support, making this a viable platform for secure IoT data streaming. See [Security Discussion](https://github.com/esp8266/Arduino/issues/43) for more information.




###Azure Event Hub

Azure Event Hubs is designed for internet scale data ingestion. 

[Stream Analytics](https://azure.microsoft.com/en-us/services/stream-analytics/), 
[Power Bi](https://powerbi.microsoft.com/en-us/) and preconfigured IoT solutions such as 
[Remote monitoring ](https://azure.microsoft.com/en-us/documentation/articles/iot-suite-remote-monitoring-sample-walkthrough) provide ways to visualise and unlock the value of your data in Azure.


###Acknowledgments
This project is a fork of [Dave Grove's project](https://github.com/gloveboxes/Arduino-ESP8266-Secure-Azure-IoT-Hub-Client), and has been slightly modified to make use of Event Hub and Different Types of Sensors, and has prior references to the below projects.
- [Proof of Concept – NodeMCU, Arduino and Azure Event Hub](https://microsoft.hackster.io/en-US/stepanb/proof-of-concept-nodemcu-arduino-and-azure-event-hub-a33043)
-[Arduino NodeMCU ESP8266 MQTT](https://github.com/gloveboxes/Arduino-NodeMCU-ESP82886-Mqtt-Client)



#Setup and Deployment Summary

1. Setup your Azure Event Hub..
2. Update the main AzureClient.ino and Publish.ino sketch
3. Deploy the solution to your ESP8266 based device.
4. View data with Azure Storage Explorer
5. Optionally: Visualise your data in real time with Azure Stream Analytics and Power BI.


#Setting Up Azure Event Hub

Follow the instructions on Azure documentations [Create an Event Hub](https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-csharp-ephcs-getstarted#create-an-event-hub) to create your own Event Hub, Access Policy and Key. We recommend creating 2 Eventhub namely "eventhubalerts" and "eventhubdevices".


#Azure Configuration


The function initCloudConfig() called from the setup() function in AzureClient.ino has two signatures. 


  
    // Inline cloud and network configuration information
    
    initCloudConfig("IoT hub device connection string", "Case Sensitive WiFi SSID", "WiFi password", "Geo location of the device"). 
   
    Example:
    
    initCloudConfig("hostname=<yourservicename>.servicebus.windows.net;SharedAccessKeyName=<yoursendername>;SharedAccessKey=<yoursharedaccesskey>", "<yourssid>", "<yourssidpassword>", "<yourlocation>);
    
    or 
    
    //To read cloud and network configuration from EEPROM. This is useful if deploying multiple devices
    
    initCloudConfig(); 
    



#Device Configuration 

You need to configure the initDeviceConfig() function in the AzureClient.ino file.


       void initDeviceConfig() {  // Example device configuration
        device.boardType = NodeMCU;            // BoardType enumeration: NodeMCU, WeMos, SparkfunThing, Other (defaults to Other). This determines pin number of the onboard LED for wifi and publish status. Other means no LED status 
        device.deepSleepSeconds = 0;         // if greater than zero with call ESP8266 deep sleep (default is 0 disabled). GPIO16 needs to be tied to RST to wake from deepSleep. Causes a reset, execution restarts from beginning of sketch
        cloud.cloudMode = EventHub;            // CloudMode enumeration: IoTHub and EventHub (default is IoTHub)
        cloud.publishRateInSeconds = 90;     // limits publishing rate to specified seconds (default is 90 seconds)
        cloud.sasExpiryDate = 1737504000;    // Expires Wed, 22 Jan 2025 00:00:00 GMT (defaults to Expires Wed, 22 Jan 2025 00:00:00 GMT)
    }   

You will also need to input your event hub name in the Publish.ino file. If you have followed the above example, you can use "eventhubdevices" below.
    
    // Azure Event Hub settings
    const char* EVENT_HUB_END_POINT ="/<youreventhubname>/publishers/sender/messages";
    
   
Then upload the sketch to your device.

#Viewing Data

If you would like to view the raw telemetry which are being sent into event hub, follow the guide below to create your own EventHostProcessor

[receive messages with eventprocessorhost](https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-csharp-ephcs-getstarted#receive-messages-with-eventprocessorhost)


#Visualising Data

##Azure Stream Analytics

[Azure Stream Analytics](https://azure.microsoft.com/en-us/services/stream-analytics/) enables you to gain 
real-time insights in to your device, sensor, infrastructure, and application data.

See the [Visualizing IoT Data](http://thinglabs.io/workshop/cs/nightlight/visualize-iot-with-powerbi/) lab.  Replace the query in that lab with the following.

    With 

    AlertInput as (SELECT Dev, 
    Min(Cast(UTC as datetime)) ,  
    Max(Celsius) as AverageTemp

    FROM Input TIMESTAMP by Cast(UTC as datetime)

    GROUP BY Dev, TumblingWindow(second, 5)

    Having AverageTemp >29
    ), StandardOutput as (
    SELECT *
    FROM Input
    )
    SELECT * INTO AlertOutput from AlertInput
    SELECT * INTO Output from StandardOutput
    
##Power BI

[Microsoft Power BI](https://powerbi.microsoft.com) makes it easy to visualise, organize and better understand your data.

Follow the notes in the See the [Visualizing IoT Data](http://thinglabs.io/workshop/cs/nightlight/visualize-iot-with-powerbi/) lab and modify the real time report as per this image.



#Data Schema

The AzureClient sketch streams data in the following JSON format, of course you can change this:)


    {"Dev":"sender","Utc":"2016-11-25T06:45:04","Celsius":24.75,"Lux":886,"Motion":0,"Gas":0,"Geo":"MS SG MIC","WiFi":1,"Mem":19016,"Id":1}

 

