---
title: "Lab 6: Closed-loop Control PID"
---

3/21/23

## Prelab
---
The first part of this lab was rethinking the way data was going to be sent back for the ToF sensor. PID control requires a constant sensor input in order to commpute the error term for the control law. This means that the current way of retrieving sensor data, in which a switch statement runs without interruption would no longer work for our purposes. This is because in the switch statement the sensor reads the data, stores it in an array, and then the array sends the data back. Instead I decided to setup my program like this pseudo code.

```code
while(true)
  sensor
  if (runPID)
    controlLaw
    recordData
  if (done)
    writeData
```
Essentially the loop runs continuously and sends power to the motor with the help of a set of flags that can be changed from the python script. I modified the robot commands used in earlier commands to just change the flags to true or false depending on if I wanted the arrays to send back data or not.

## Lab Tasks

I initially started out with just a P controller to easily get something that could get close to the end result. I first examined the control loop. My reference value was approximately 300 mm. The distance sensor would measure up to a max value of 4000 mm. This meant my minimum error was 0 and my maximum possible error was -3700. Next I worked to find the control law. In this case the motor can be set to any value between [0 255]. However, because of the deadband of the motor which we examined in the last lab, a value up to 40 PWM does not cause any wheel rotation. So, I decided to have my minimum PWM be 40 and my max be 200. But to map my control U = Kp * E, onto those values, I mapped [0 3700] onto [0 160] and then added 40 to the result to have a U > 0 cause wheel rotation. I set my Kp = 160/3700 = 0.043. This meant my final control law was U = Kp * E + 40.

Block Diagram:

![image](https://user-images.githubusercontent.com/123790450/226800116-55d2c02e-c32f-4b23-9d69-b8e93a17c488.png)

I decided to stick with the long range sensing value on the distance sensor because it would allow me to have the robot start from a further away position better. Additionally it was able to measure 1 foot accurately to +- 5 mm in my testing from a previous lab. The robot samples data as quickly as it comes in. I initially tried to not check for data ready however it would lead to the distance sensor value getting stuck to one value for the rest of the loop causing the robot to crash into the wall no matter how close the actual value was.

```c++
while (!distanceSensor2.checkForDataReady()) {
        delay(1);
      }
```

