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


As the device is rotated, the IMU registers the gravity force vector in different directions relative to it's own orientation. When upright, it see it as a postive value of 1000. When upsdie down it sees the same force as -1000. When on the side it sees it as a negative value in the y direction. The IMU has a small unit vector diagram (x,y,z) showing which direction the positive force is in.
![image](https://user-images.githubusercontent.com/123790450/220525407-09663ecc-7d00-498a-96fe-e62fe38741d0.png)


## Accelerometer

To calculate pitch and roll two equations can be used along with the output of the accelerometer. The formula for pitch is: ![image](https://user-images.githubusercontent.com/123790450/220526373-0948dfe9-bca9-4f6b-9e71-745523c2cd43.png). The formula for roll is: ![png](https://user-images.githubusercontent.com/123790450/220526776-43d63b79-9445-4137-8dbe-77863381d06c.png)

Roll -90: ![image](https://user-images.githubusercontent.com/123790450/220528150-445b810c-eaef-426f-8a3d-06dfd630fa1a.png)

Roll 0: ![image](https://user-images.githubusercontent.com/123790450/220528803-a870597e-b68c-4541-b2e8-c67c04930d7c.png)








