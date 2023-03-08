---
title: "Lab 5: Motor Controller and Open Loop Control"
---
Date: 3/7/23

In this lab we swap the RC car's motor drivers with our own.

## Setup

Here is schematic showing how the connections look from the Artemis to the motor driver, motor, and battery:

![image](https://user-images.githubusercontent.com/123790450/223622419-db7ac800-2c08-4aae-a149-d7b321f34376.png)

To connect a motor driver it is important to use analog PWM pins to send power that is not just high or low. I used the [schematic](https://cdn.sparkfun.com/assets/5/5/1/6/3/RedBoard-Artemis-Nano.pdf) in order to figure out which pins were PWM. I decided to use pins A2,A3,A4,and A5 as my motor driver pins. Additionally in order to provide more current to each motor we parallel couple the input and output of the motor driver as seen in the wiring diagram because it is capable of controlling two separate motors.


