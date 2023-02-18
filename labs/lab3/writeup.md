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
Something interesting occured while scanning for I2C devices. The address show is Ox29 instead of the expected Ox25. The reason for this is that 0x29 is just 0x52 with one bit used to know if the devices is read or writing instead of for the address. 0x52 in binary is 0b1010010 while 0x29 0b101001. 0x29 is just 0x52 with a right bit shift.
