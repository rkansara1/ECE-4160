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

Roll 90: ![image](https://user-images.githubusercontent.com/123790450/220529049-7ad258c4-df18-4220-b317-82be4385fa77.png)

Pitch -90: ![image](https://user-images.githubusercontent.com/123790450/220529172-bfee2639-7d42-4498-bea9-93477c002a98.png)

Pitch 0: ![image](https://user-images.githubusercontent.com/123790450/220529228-b4912ee5-30d6-4bf2-a5b7-43a36371ad01.png)

Pitch 90: ![image](https://user-images.githubusercontent.com/123790450/220529245-a4b92ae5-1d60-4371-a390-ed7d136046c0.png)

When performing tests by placing the IMU at -90 and 90 degrees the output was almost exactly the angle the IMU was at. Due to this I did not implement a scaling factor that would fit the data to a range.

I performed a FFT in Python on the pitch and roll data:

![image](https://user-images.githubusercontent.com/123790450/220535093-33d19f02-f20d-454d-9384-b6de7113c1d4.png)

![image](https://user-images.githubusercontent.com/123790450/220535179-249f5a00-cd4a-4dc8-9afd-215fc68596e6.png)

After analyzing these two fourier transforms there isn't much activity on the higher frequency end of the spectrum, so it seems like implementing a low pass filter won't be necessary.

## Gyroscope
To calculate any of the angles using gyroscope data a simple formula is used: ![image](https://user-images.githubusercontent.com/123790450/220538074-bb565949-d77b-493d-b8d7-c58d4c8bf9bb.png). This formula is then implemented in the Arduino like this:

```c++
      dt = (micros() - last_time) / 1000000.;
      last_time = micros();
      pitch_g = pitch_g + myICM.gyrY() * dt;
      roll_g = roll_g + myICM.gyrX() * dt;
      yaw_g = yaw_g + myICM.gyrZ() * dt;
```
[NOTE: Sourced from Lecture 4 Code]

The video shows the output of the three angles from the gyroscope data using the above formulas:

<video src="https://user-images.githubusercontent.com/123790450/222938654-6e765a9f-09f9-4421-ba81-7d22adc7f6ef.webm" controls="controls" style="max-width: 730px;">
</video>

Next I compared the accelerometer data to the gyroscope data to see how they differed. The main difference was the gyroscope data had considerably less noise but it also tended to drift.

Pitch Comparison:
![image](https://user-images.githubusercontent.com/123790450/220537636-47589950-5472-43a3-9ed3-e9439cf1c2b4.png)

Roll Comparison:
![image](https://user-images.githubusercontent.com/123790450/222938921-71e8d655-afad-4e24-b4ca-9f28ab2df099.png)











