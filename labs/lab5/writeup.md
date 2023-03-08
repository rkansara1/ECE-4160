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

I then soldered the motor leads onto each motor controller. My next test was to see if I could get the robot to spin on its side. Still using the power supply I tested rotating each side of the wheels using the same code as before. I also tested going clockwise and counterclockwise with the motors. To accomplish that in the program I just changed which pin I was writing to. So I executed the previous code or this code to test.
```c++
  analogWrite(2, 75);
  analogWrite(3, 0);
  analogWrite(4, 0);
  analogWrite(5, 75);
  ```

<video src="https://user-images.githubusercontent.com/123790450/223635040-1c92fe3c-9338-47fd-b0ff-df26afd08459.MOV" controls="controls" style="max-width: 730px;">
</video>

My next step was soldering the battery onto the power connections for the motor controller to run both of the motors at once without the power supply. I executed both of the previous code snippets to get the following result. The right set of wheels were spinning slower but that was not an actual issue with the motors that was just due to a low PWM value.

<video src="https://user-images.githubusercontent.com/123790450/223635845-30f1b32d-a4bf-4a8c-902f-203285491aa7.MOV" controls ="controls" style="max-width: 730px;">
</video>

With all the connections confirmed working I now focused on securing all the components adequately to the car. The following diagram shows how I secured the components and where I placed all the different sensors/controllers.

<img width="1409" alt="image" src="https://user-images.githubusercontent.com/123790450/223640069-882e9967-f45c-4517-8a66-a7842518ed61.png">

Next I experimentally tested to find the dead band in PWM for my motors. Through incrementing through different values I found the dead band for my motor.

```c++
  analogWrite(2, 0); // LEFT
  analogWrite(3, 39);
  analogWrite(4, 28);//RIGHT
  analogWrite(5, 0);
```
The motor driving the left wheels had a higher minimum PWM at 39 compared to the right motor at 28. Here is a video showing the motor stalling.

<video src="https://user-images.githubusercontent.com/123790450/223639737-ed27f8e5-2de4-4ad4-af0e-f7b57a1fafba.MOV" controls="controls" style = "max-width: 730px;"> </video>


Next I placed the motor on the ground to see if any motor calibration was needed.


https://user-images.githubusercontent.com/123790450/223636208-bdcced6e-665f-4b5c-a4fd-cf6a21f8121b.MOV

