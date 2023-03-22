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
  controlLaw
  recordData
  if (done)
    writeData
```
Essentially the loop runs continuously

## Lab Tasks
