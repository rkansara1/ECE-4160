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
  if (error < threshold)
    done = true
    runPID = false
  if (done)
    writeData
```
Essentially the loop runs continuously and sends power to the motor with the help of a set of flags that can be changed from the python script. I modified the robot commands used in earlier commands to just change the flags to true or false depending on if I wanted the arrays to send back data or not.

## Lab Tasks
---

I initially started out with just a P controller to easily get something that could get close to the end result. I first examined the control loop. My reference value was approximately 300 mm. The distance sensor would measure up to a max value of 4000 mm. This meant my minimum error was 0 and my maximum possible error was -3700. Next I worked to find the control law. In this case the motor can be set to any value between [0 255]. However, because of the deadband of the motor which we examined in the last lab, a value up to 40 PWM does not cause any wheel rotation. So, I decided to have my minimum PWM be 40 and my max be 200. But to map my control U = Kp * E, onto those values, I mapped [0 3700] onto [0 160] and then added 40 to the result to have a U > 0 cause wheel rotation. I set my Kp = 160/3700 = 0.043. This meant my final control law was U = Kp * E + 40.

Block Diagram:

![image](https://user-images.githubusercontent.com/123790450/226800116-55d2c02e-c32f-4b23-9d69-b8e93a17c488.png)

I decided to stick with the long range sensing value on the distance sensor because it would allow me to have the robot start from a further away position better. Additionally it was able to measure 1 foot accurately to +- 5 mm in my testing from a previous lab. The robot samples data as quickly as it comes in. I initially tried to not check for data ready however it would lead to the distance sensor value getting stuck to one value for the rest of the loop causing the robot to crash into the wall no matter how close the actual value was. I am not initially attemtping an Integrator or Derivative term so the speed of the sampling doesn't matter as much as it would in those scenarios.

```c++
while (!distanceSensor2.checkForDataReady()) {
        delay(1);
      }
```

Results:

<video src = "https://user-images.githubusercontent.com/123790450/226801282-d7b63d24-c471-4f30-b238-2b82d8a8940d.mov" controls = controls style="max-width:730px;"></video>

**UPDATE**:
---
I was told the video was not working, so here is an uploaded copy of the above video on youtube.
<iframe width="560" height="315" src="https://www.youtube.com/embed/CiIaTM7zUOc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

Based on the video the robot is able to complete the task successfully. It look like it gets within a half inch (~13 mm) of the goal.


![y step response trial 3](https://user-images.githubusercontent.com/123790450/226803500-243590a7-03e0-4114-8d13-c89e15968923.png)

This graph shows the step response of the ouput to a step change in input of 300 mm. The robot slightly overshoots the goal of 300 mm but corrects itself. This system is underdamped but still reaches steady state quickly with it only taking approximately 3.7 seconds. There is some initial bug in the distance sensor reading but it quickly corrects itself and behaves normally.

![motor pwm trial 3](https://user-images.githubusercontent.com/123790450/226803531-cf027858-f9c7-45e5-b0bf-5815c6b7342c.png)

The motor input initially does not start at a large value because of a misreading of the distance sensor. However once the distance sensor updates correctly the PWM signal is large. and then decreases linearly until it goes past zero because of overshoot. The negative PWM value in this case represents going forwards while the positve PWM values represent going backwards. This value doesn't include the 40 PWM that is added to avoid deadband because of the way the value was implemented. Adding the deadband PWM depends on the direction the motor is spinning so adding 40 to the control signal can have the effect of making the motor move the wrong way.

