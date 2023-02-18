---
title: "Lab 3: Time of Flight Sensors"
---

## Prelab

My robot will have two Time of Flight Sensors on it. To allow the use of two time of flight sensors with the same I2C address one of the sensors just needs to have its I2C address changed from the default of Ox52. To do this I will take advantage of the `XSHUT` pin on the sensor. If the `XSHUT` pin is set to low on of the ToF sensors, then it is possible to change the I2C address programatically on the other sensor and then turn the `XSHUT` pin back to high to have two working ToF sensors.

In order to ensure I have the best view possible I plan on putting one ToF sensor on the front of the robot and one on the side of the robot (as seen in the below image). This way I should be able to tell what is in front of me and what is to the side of me. Additionally by rotation 180 degrees I can capture a full 360 degree field of obstacles. A scenario I can miss an object is in the beginning with where my robot gets placed. If an obstacle is in the blindspot of the robot then I won't be able to see it initially and might not be able to make the full 180 degree turn to capture obstacles.

<img width="515" alt="image" src="https://user-images.githubusercontent.com/123790450/219848689-880f7746-33de-4246-abc5-9a311a87238b.png">

Wiring Diagram:

![image](https://user-images.githubusercontent.com/123790450/219849323-3d00c0a3-a3c9-45fd-9bfc-aea1e813e9e7.png)
Note: `XSHUT` is connected to `PIN 8`


## Lab Tasks

ToF Sensor Connected to QWIIC Breakout Board:
![IMG_1007](https://user-images.githubusercontent.com/123790450/219849507-41d61cb9-59ab-4d6d-ab12-1914e69c0c50.jpg)

Artemis Scanning for I2C Device:

![image](https://user-images.githubusercontent.com/123790450/219849611-c7c522bd-9918-4017-9ac8-4b94a4bf77ba.png)
Something interesting occurred while scanning for I2C devices. The address show is Ox29 instead of the expected Ox25. The reason for this is that 0x29 is just 0x52 with one bit used to know if the devices is read or writing instead of for the address. 0x52 in binary is 0b1010010 while 0x29 0b101001. 0x29 is just 0x52 with a right bit shift.

Sensor Mode:
The ToF sensors have three different distance modes that can be chosen from. They are:
![image](https://user-images.githubusercontent.com/123790450/219849870-7929639e-1f39-45f3-ac8c-040232382499.png)

A function in the included library allows to easily switch between modes. The medium distance mode is not able to be set from the library however. There is only a `setDistanceModeShort()` and `setDistanceModeLong()`. After performing a test by placing a piece of paper and seeing how accurate the sensor was in either mode, I was able to conclude that the long mode suited my situation better. Additionally in terms of the robots performance being able to see the furthest out will help the most with planning routes to take rather than just having higher fidelity nearby distances.

Using 11.5" benchmark:

![image](https://user-images.githubusercontent.com/123790450/219849972-ff945a06-76f8-47b4-8cf2-db638537452b.png)

Distance Mode Long:

![distanceModeLong](https://user-images.githubusercontent.com/123790450/219849984-6591ce80-b2c2-4cb7-b912-ed027df44211.png)

Distance Mode Short:

![distanceModeShort](https://user-images.githubusercontent.com/123790450/219849987-8206030b-b8b8-4687-b769-8242588e42b1.png)


### 2 ToF Sensors
Using the strategy discussed in Prelab it was very straightforward to get two ToF sensors working together. In the void setup all that needed to be included was:
```c++
 digitalWrite(SHUTDOWN_PIN, LOW);
  distanceSensor1.setI2CAddress(10);
  digitalWrite(SHUTDOWN_PIN, HIGH);
```
Two ToF Sensors Connected:

![image](https://user-images.githubusercontent.com/123790450/219850154-4e04aa97-cb9f-4290-92c3-c708930cc545.png)




