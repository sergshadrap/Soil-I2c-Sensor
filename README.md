# Serg-S
Mpp-I2C-Soil_sensor
MppSoilMoistureSleeper
The board has designed for working with Mike Polan's software Am Server(AM Manager) https://sites.google.com/site/mppsuite/.
It intended to measure Soil humidity and temperature.

 Is-an MppSleeper (battery powered or solar powered) to read a sensor I2C soil moisture meter (https://www.tindie.com/products/miceuz/i2c-soil-moisture-sensor/  has made by Miceuz from Catnip electronics.) with ESP8266 (12F) soldered on my design PCB board.
 
 it wakes periodically to report the sensor value.
 wifi is disabled until needed


 Monitor the device from AM to detect when it fails to report.  One rule per device is needed:
 Trigger:  OnEvents [yourIpReportValue] interruptible
 OnStart // to restart the wait if you edit rules
 Wait: SleepTime * ReportCount * (MeasureTime / 60) * 1.1 minutes (1.1 gives some range for timing skews)
 Action:  SendGMAIL about a missed report


Device wakes on an RST ,take digital measurment from I2C sensor, report the values of soils temperature,relative humidity and battery voltage.
 If the device home page is hit (in either mode) it will reset the time till sleep to 5 minutes.

 WifiPin - if set will show the wake status
 WifiActive - pick to show LED when awake
 ServerIp - the IP of the target server for the wake message (port 8898)
 IpReport - the AM Server OnEvent to send for a periodic check in
 MaxSleepTime - max deep sleep in minutes, if not available assume 70 minutes
 SleepTime - deep sleep in minutes, max as reported in MaxSleepTime
 ReportCount - number of deep sleep cycles before sending the report message (avoid enabling wifi)
 NoMulticast - if ServerIp is set this property is set to true to avoid double notifications
 AwakeTime - the time in seconds the device will remain awake for a wifi connection
 MeasureTime - warmup time before measurement if SensorEnablePin is used
 SensorEnablePin - pin to enable or power the sensor
 KIND_SOIL- Kind of soil (if soil tends to be more sandy, more loamy or more clay). 
 
 To wake the device for configuration, trigger the wake pin and hit it with a browser as it loads (within 10s)
 You'll have 5 minutes more after each browser refresh to configure it.

How it works:
The device started from measurement mode with RF disabled, take measurement from I2C sensor, stores it in RTC memory and reboots in WIFI mode, 
Sends data to AM server and fall a sleep for time "SleepTime".
TP4056(solar charger controller) the board which responsible for charging a battery,additionally the connection used GPIO12 and GPIO14 reserved as charging control. 

GPIO13 is used by default for sensor powering.

The sensor using capacitive kind of measurement around the sensor probe.( consequently there is no constant current along the sensor and no electrolysis.)
Depending on that value it calculates absolute water(Wa) capacity of current place of soil.
Wa=(Mw-Md)/Md*100% (where Mw- a mass of moistured soil and Md- a mass of dryed soil). Relative soil humidity (Wr) is calculated as Wr= Wa/Wn*100% (where Wn- is Fields water capacity coefficient)
Due to high stage non homogeneous structure the soil has different ability to keep the water inside. This dependency is expressed in three major coefficient for different kinds of soil - sandy,loam and clay.
(Frankly the dependance is much more complicated but for simplicity i'm using only three coefficients.)
The typical results are : Wr<20% - critically low humidity; 20%-50%- generally dry soil; 50%-80% good moisturized soil; 80-100% - too much water; 110% - swamp.
The sensor doesn't intended to measure soil humidity with high accuracy - humidity measurement has about 15% of accuracy, temperature about 5%.
For better quality of measurement you have to provide more better contact with soil. 
For outdoor using it is make a sense to bury the sensor probe to the level of plant's roots.
The measurement are very voltage dependable.Due to battery discharging the figures might changing dramatically.For this issue i'm using two lagrange approximations.
I have plot two graphs - the first empiric values of capacity changes during the voltage, the second - soil humidity evaluation through the capacity and kind of soil.
But still... the more stable power gives more stable results. That's why i'm using MCP1700 LDO in my circuit.
I still have to rack my brain over temperature compensation , but turns out that it highly correlated with real soil structure and water content.

Battery_Voltage - Mpp analog device for battery voltage reporting. (device name UDN+_B)
Soil_Humidity -   Mpp analog device for soil humidity reporting. (device name UDN+_H)
Soil_Temperature  - Mpp analog device for bsoil temperature reporting. (device name UDN+_T)
 Overall current consumption : deep sleep mode - about 30-50ua (20ua in battery config), measurment(no wifi) about 20ma, wifi mode (data transmission) about 75-85ma.
It works about a year on 3000ma battery w\o solar charging.
 In solar panel version :
 Typically i'm using 80x80 mm solar panel that gives about 5-80ma charging current in sunny day (for Solar irradiance in my region) and LION TR18650 3500mAh 3.7V battery.
For the battery version you don't need any LDO , TP4056 charger controller or solar pannel. Just only 2 AA 1.5V battery and ESP8266 .

Sergey S. shadrap@yandex.ru .
