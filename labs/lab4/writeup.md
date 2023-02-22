---
title: "Lab 4: IMU"
---
2/21/23

In this lab we are going to set up an IMU to be able to track our robots acceleration as well as pitch, roll, and yaw.
## Setting up the IMU
To first set up the IMU. The breakout board needs to be connected to the Artemis through the Quicc connector like in the below image.

![imu_connected_small](https://user-images.githubusercontent.com/123790450/220521804-531cd9f8-dd16-408c-9369-dc5646ea78e6.jpg)

Next I ran the sketch found in File>Examples>Sparkfun 9DOF...>Example1_Basics.ino. Below is the output of that sketch.

![image](https://user-images.githubusercontent.com/123790450/220520739-5481df23-b353-4180-a6ba-e6ec43c4c304.png)

The `AD0_VAL` is what is used to connect multiple IMUs to the Artemis. When `ADO_VAL` is set to high, the I2C address is 0x69, when value is set to low the I2C address is 0x68. This allows the use of two IMUs on the same I2C with different addresses. This is similar to the way we setup the ToF sensors. In our case though, we only have 1 IMU so I left the value to it's default of high.

This is what the IMU data looks like when the board is flat on my desk:

![image](https://user-images.githubusercontent.com/123790450/220524743-48a758b9-e33c-46e7-93e1-bad823bde36c.png)

This is what the IMU data looks like when the board is rotated 180 degrees to be face down on my desk:

![image](https://user-images.githubusercontent.com/123790450/220524822-b7e37c8e-7483-4e1a-b32f-63106a75a06b.png)

This is what the IMU data looks like when it is rotated on it's side:

![image](https://user-images.githubusercontent.com/123790450/220524886-738e159d-7658-47a2-877d-75979001677f.png)


