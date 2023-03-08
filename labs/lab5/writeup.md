---
title: "Lab 5: Motor Controller and Open Loop Control"
---
Date: 3/7/23

In this lab we swap the RC car's motor drivers with our own.

## Setup

Here is schematic showing how the connections look from the Artemis to the motor driver, motor, and battery:

![image](https://user-images.githubusercontent.com/123790450/223622925-f5dbd90b-e19b-43e7-b718-579dd6ecf059.png)

To connect a motor driver it is important to use analog PWM pins to send power that is not just high or low. I used the [schematic](https://cdn.sparkfun.com/assets/5/5/1/6/3/RedBoard-Artemis-Nano.pdf) in order to figure out which pins were PWM. I decided to use pins A2,A3,A4,and A5 as my motor driver pins. Additionally in order to provide more current to each motor we parallel couple the input and output of the motor driver as seen in the wiring diagram because it is capable of controlling two separate motors.

The motors and the Artemis are powered by separate batteries to avoid EMI issues. By having separate power sourcses we can minimize the noise from the motors and motor drivers to our IMU and ToF sensors.

## Lab Tasks

I first initially tested the system by hooking up a power supply set to 3.7 V (the same voltage as the battery) to the `VIN` and `GND` pins on each motor controller. I then also connected the output of the motor controller to an oscilloscope to read the signal output. I did this to test if my initial wiring of the motor controllers was accurate on the input side. Here is a snippet of the code I ran to generate the oscilloscope signal
```c++
  analogWrite(2, 0);
  analogWrite(3, 75);
  analogWrite(4, 75);
  analogWrite(5, 0);
  ```

![image](https://user-images.githubusercontent.com/123790450/223633756-9062ed6a-4b2f-459f-ba68-dd764ab70137.png)
![image](https://user-images.githubusercontent.com/123790450/223634407-c273f8b2-0106-488d-9760-967e8de8b3ec.png)

