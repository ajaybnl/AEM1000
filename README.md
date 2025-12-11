# Reading AEM1000 for Arduino Projects
AEM1000 Sensor Reading Methord for Arduino , ESP32 , ESP8266 Like MCU

## Purpose
I have made a cheap multi Air sensors project using ESP8266 , 1.8" TFT. 
I have used Node-red to upload the sensor data to mysql db from Mqtt.


## Info on Board
The ASAIR AEM1000 is a cheap alternative. The data it provides is calibrated from factory. We can fetch the data by modbus protocol either from Uart or modbus module.

**You have to transfer 2 resistors and remove 2 big resistors to use UART**
**See Uploaded Photo**

Datasheet: http://www.aosong.com/m/en/products-78.html

I was new to this protocol, and not found a library for this sensor. 
I have chosen this library for this sensor : 
https://github.com/4-20ma/ModbusMaster

## Some Insights



```cpp
//Note: Defaultly there is only MODBUS protocol supported on AEM1000
// You have to transfer 2 resistors as shown in image in the main page

// Create SoftwareSerial instance
SoftwareSerial modbusSerial(D1, D2);  // RX, TX pins
ModbusMaster node;  // Create Modbus master instance

//Setup Code
 modbusSerial.begin(9600);
  node.begin(1, modbusSerial);
```

## Reading Sensor Values

```cpp
//The AEM1000 sensors registers are from 0-4 and size max is 4

const char *names[] = { "temp", "hum", "voc", "co2", "pm2.5" };

    for (int i = 0; i <= 4; i++) {

      result1 = node.readHoldingRegisters(i, 4);

      if (result1 == node.ku8MBSuccess) {
        data1 = node.getResponseBuffer(0);

//store sensor values in values[] array
values[i] = (int)data1;

//Print senor name and value
Serial.print(names[i]);
Serial.print(" : "):
Serial.println(values[i]);
      } else {
   //error fetching sensor data or corrupt data
      }
     
    }
```

## To Calibrate Co2 (keep sensor in open air outside for 20 minutes and run this)

```cpp

//set co2 calibration to manual
 node.writeSingleRegister(32, 0);

//set value 405 as co2 value of open area (natural air)
    node.writeSingleRegister(33, 405);
```


## To Check Sensor Status (If Sensor Working or faulty or unplugged)


```cpp

const char *names[] = { "temp", "hum", "voc", "co2", "pm2.5" };

//The registers are from 11-14 (for same sensors register from 0-4)

 for (int i = 11; i <= 14; i++) {
    
      result1 = node.readHoldingRegisters(i, 4);

      if (result1 == node.ku8MBSuccess) {

       status1 = node.getResponseBuffer(0);

       // Status : 0:OK, 1:Faulty, 3:Unplugged

      const char *statuses[] = {"OK","Faulty","Unplugged"};
    

        // Print Status
         Serial.print (names[i - 11]);
         Serial.print(" : ");
        Serial.println(statuses[status1]);     
        
      } else {

        Serial.println("Error reading register");
      }

    }
```

## Feel free to use it in your code



