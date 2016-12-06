#Securely stream data from ESP8266 MCUs to Azure IoT Hub over HTTPS/REST

###References
This project is a fork of [Dave Grove's project](https://github.com/gloveboxes/Arduino-ESP8266-Secure-Azure-IoT-Hub-Client), and has been slightly modified to make use of Event Hub and Different Types of Sensors.

###Purpose

This solution securely streams sensor data directly to 
[Azure Event Hub](https://azure.microsoft.com/en-us/services/event-hubs/) over HTTPS calling Azure REST APIs from ESP8266 MCUs.


###Device Platform


The [ESP8266](https://en.wikipedia.org/wiki/ESP8266) is a device based out of Espressif ESP8266 which comes with integrated WiFi. 

This project has been tested on the following ESP8266 based development boards:-

1. [NodeMCU V2 (also known as V1.0)](https://en.wikipedia.org/wiki/NodeMCU)


###Firmware

[Arduino core for ESP8266 WiFi chip V2.0](https://github.com/esp8266/Arduino) firmware adds HTTPS ([TLS](http://axtls.sourceforge.net/) 1.0 and 1.1) support, making this a viable platform for secure IoT data streaming. See [Security Discussion](https://github.com/esp8266/Arduino/issues/43) for more information.




###Azure Event Hub

Azure Event Hubs is designed for internet scale data ingestion. 

[Stream Analytics](https://azure.microsoft.com/en-us/services/stream-analytics/), 
[Power Bi](https://powerbi.microsoft.com/en-us/) and preconfigured IoT Hub solutions such as 
[Remote monitoring ](https://azure.microsoft.com/en-us/documentation/articles/iot-suite-remote-monitoring-sample-walkthrough) provide ways to visualise and unlock the value of your data in Azure.

For more background information read this "[Stream Analytics & Power BI: A real-time analytics dashboard for streaming data](https://azure.microsoft.com/en-us/documentation/articles/stream-analytics-power-bi-dashboard/)" article.

###Acknowledgments
This project is a fork of [Dave Grove's project](https://github.com/gloveboxes/Arduino-ESP8266-Secure-Azure-IoT-Hub-Client), and has been slightly modified to make use of Event Hub and Different Types of Sensors, and has prior references to the below projects.
- [Proof of Concept – NodeMCU, Arduino and Azure Event Hub](https://microsoft.hackster.io/en-US/stepanb/proof-of-concept-nodemcu-arduino-and-azure-event-hub-a33043)
-[Arduino NodeMCU ESP8266 MQTT](https://github.com/gloveboxes/Arduino-NodeMCU-ESP82886-Mqtt-Client)



#Setup and Deployment Summary

1. Setup your Azure Event Hub.
2. Register your device with Azure IoT Hub.
3. Update the main AzureClient.ino sketch
4. Deploy the solution to your ESP8266 based device.
5. View data with Azure Storage Explorer
6. Optionally: Visualise your data in real time with Azure Stream Analytics and Power BI.


#Setting Up Azure IoT Hub

Follow the instructions on Azure documentations [Create an Event Hub](https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-csharp-ephcs-getstarted#create-an-event-hub) to create your own Event Hub, Access Policy and Key


#Azure Configuration


The function initCloudConfig() called from the setup() function in AzureClient.ino has two signatures. 


  
    // Inline cloud and network configuration information
    
    initCloudConfig("IoT hub device connection string", "Case Sensitive WiFi SSID", "WiFi password", "Geo location of the device"). 
   
    Example:
    
    initCloudConfig("hostname=<yourservicename>.servicebus.windows.net;SharedAccessKeyName=<yoursendername>;SharedAccessKey=<yoursharedaccesskey>", "<yourssid>", "<yourssidpassword>", "<yourlocation>);
    
    or 
    
    //To read cloud and network configuration from EEPROM. This is useful if deploying multiple devices
    
    initCloudConfig(); 
    


##Optional EEPROM Configuration

You can burn cloud and network configuration information to the device EEPROM.  To configure the EEPROM open the SetEEPROMConfiguration.ino found in the SetEEPROMConfiguration folder and update the following variables:-

  - Wi-Fi SSID and password pairs, put in priority order.
  - Wifi - Is the number of WiFi ssid and password pairs
  - Azure IoT Hub or Event Bus Host name eg "MakerDen.azure-devices.net", Device ID, and Key. For IoT Hub get this information from the Device Explorer, for Event Hub, get from Azure Management Portal.
  - Geo location of the device
  
Upload this sketch to the device to write configuration settings to EEPROM.

**Be sure to call function initCloudConfig() with no parameters to ensure Azure and Network parameters are read from EEPROM.**


Next upload the AzureClient sketch which will 
read this configuration information from the EEPROM. 


#Device Configuration 

You need to configure the initDeviceConfig() function in the AzureClient.ino file.


       void initDeviceConfig() {  // Example device configuration
        device.boardType = NodeMCU;            // BoardType enumeration: NodeMCU, WeMos, SparkfunThing, Other (defaults to Other). This determines pin number of the onboard LED for wifi and publish status. Other means no LED status 
        device.deepSleepSeconds = 0;         // if greater than zero with call ESP8266 deep sleep (default is 0 disabled). GPIO16 needs to be tied to RST to wake from deepSleep. Causes a reset, execution restarts from beginning of sketch
        cloud.cloudMode = EventHub;            // CloudMode enumeration: IoTHub and EventHub (default is IoTHub)
        cloud.publishRateInSeconds = 90;     // limits publishing rate to specified seconds (default is 90 seconds)
        cloud.sasExpiryDate = 1737504000;    // Expires Wed, 22 Jan 2025 00:00:00 GMT (defaults to Expires Wed, 22 Jan 2025 00:00:00 GMT)
    }   

You will also need to input your event hub name in the Publish.ino file.  
    
    // Azure Event Hub settings
    const char* EVENT_HUB_END_POINT ="/<youreventhubname>/publishers/sender/messages";
    
   
Then upload the sketch to your device.

#Viewing Data

From Device Explorer, head to the Data tab, select your device, enable consumer group then click Monitor.

![IoT Hub Data](https://raw.githubusercontent.com/gloveboxes/Arduino-NodeMCU-ESP8266-Secure-Azure-IoT-Hub-Client/master/AzureClient/Fritzing/IoTHubData.JPG)


#Visualising Data

##Azure Stream Analytics

[Azure Stream Analytics](https://azure.microsoft.com/en-us/services/stream-analytics/) enables you to gain 
real-time insights in to your device, sensor, infrastructure, and application data.

See the [Visualizing IoT Data](http://thinglabs.io/workshop/cs/nightlight/visualize-iot-with-powerbi/) lab.  Replace the query in that lab with the following and be sure to change the time zone to your local time zone offset.  Australia (AEDST) is currently +11 hours.

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

###Power BI Designer Setup

![Power BI Designer Setup](https://raw.githubusercontent.com/gloveboxes/Arduino-NodeMCU-ESP8266-Secure-Azure-IoT-Hub-Client/master/AzureClient/Fritzing/PowerBIDesigner.JPG)


###Power BI Report Viewer

View on the web or with the Power BI apps available on iOS, Android and Windows.

![Power BI Report Viewer](https://raw.githubusercontent.com/gloveboxes/Arduino-NodeMCU-ESP8266-Secure-Azure-IoT-Hub-Client/master/AzureClient/Fritzing/PowerBIReport.JPG)



#Data Schema

The AzureClient sketch streams data in the following JSON format, of course you can change this:)


    {"Dev":"sender","Utc":"2016-11-25T06:45:04","Celsius":24.75,"Lux":886,"Motion":0,"Gas":0,"Geo":"MS SG MIC","WiFi":1,"Mem":19016,"Id":1}

 
#Software Requirements

##Drivers

1. NodeMCU - On Windows, Mac and Linux you will need to install the latest [CP210x USB to UART Bridge VCP Drivers](https://www.silabs.com/products/mcu/Pages/USBtoUARTBridgeVCPDrivers.aspx).
2. WeMos - On Windows and Mac install the latest [Ch340G drivers](http://www.wemos.cc/Tutorial/get_started_in_arduino.html). No drivers required for Linux.
3. [ESP8266 Thing Development Board Hookup Guide](https://learn.sparkfun.com/tutorials/esp8266-thing-development-board-hookup-guide/hardware-setup)


##Arduino IDE

As at April 2015 use:-

1. [Arduino IDE 1.6.8](https://www.arduino.cc/en/Main/Software).
2. ESP8266 Board Manager 2.1 or better required for HTTPS/TLS Secure Client support.

##Visual Studio

There an fantastic plugin for Visual Studio that adds Arduino support from [Visual Micro](http://www.visualmicro.com/).  IntelliSence, auto complete, debugging, it doesn't get much better:)

##Arduino Boards Manager ESP8266 Support

Arduino version 1.6.5 and above allows installation of third-party platform packages using Boards Manager.

1. Install Arduino 1.6.5 from the Arduino website.
2. Start Arduino and open Preferences window.
3. Enter http://arduino.esp8266.com/stable/package_esp8266com_index.json  into Additional Board Manager URLs field. You can add multiple URLs, separating them with commas.
4. Open Boards Manager from Tools > Board menu and install esp8266 platform (and don't forget to select your ESP8266 board from Tools > Board menu after installation).
5. Select NodeMCU or WeMos D1 Mini Board: Tools -> Board -> NodeMCU 1.0 (ESP-12E module) or WeMos D1 Mini
6. Set Port and Upload Speed: Tools.  Note, you may need to try different port speeds to successfully flash the device. Faster is better as each time you upload the code to your device you are re-flashing the complete ROM not just your code.

#ESP8266 Arduino Core Documentation 

Be sure to read the [ESP8266 Arduino Core Documentation](http://esp8266.github.io/Arduino/versions/2.0.0/) - there are some minor gotchas.
