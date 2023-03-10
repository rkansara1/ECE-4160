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

To use both sources of data I implemented a complementary filter. To implement the complementary filter I used a formula from this [pdf](https://docs.google.com/viewer?a=v&pid=sites&srcid=ZGVmYXVsdGRvbWFpbnxteWltdWVzdGltYXRpb25leHBlcmllbmNlfGd4OjY1Yzk3YzhiZmE1N2M4Y2U). The formula calculates the weight of the complementary filter for a desired time constant. ![image](https://user-images.githubusercontent.com/123790450/222941725-4fc98e8b-18ed-45bd-aa5c-1773417e1223.png). I decided I wanted my filter to have a time constant of approximatley 0.5 seconds. This was primarily so that I could have the filter use the gryoscope data for short term accuracy and the acceleromater data for long term accuracy. This would allow the accelerometer to correct the gyroscope drift and would filter out the accelerometer noise. After printing out the time for the loop to complete, I was able to calculate a dt of about 0.00185 seconds. Using this value I could now calculate alpha. ![image](https://user-images.githubusercontent.com/123790450/222942000-e0529cce-8555-41a6-acfd-e3d954f8efd5.png)



```c++
      float alpha = 0.996;
      float beta = 0.996;
      pitch = (pitch - myICM.gyrY()*dt) * alpha + pitch_a * (1-alpha);
      roll = (roll + myICM.gyrX()*dt) * beta + roll_a * (1-beta);
```

Pitch with complementary filter:

![image](https://user-images.githubusercontent.com/123790450/222942142-e671efa4-6e40-4331-88e3-a749a8e00d3a.png)

Roll with complementary filter:

![image](https://user-images.githubusercontent.com/123790450/222942163-d9749881-0961-479c-9562-fda69b313878.png)

Based on the results the filter is able to track the two desired values with little noise while also combatting drift. The chosen weight is adequate for robot use.

## Sample Data
First I evalued how fast my sensor was able to sample data. To accomplish this I set up a simple loop that just measured the time it took to collect one set of data from the IMU. After running the following code I found it took 0.3 seconds to collect data. This was longer than the 80ms it took to get the ToF sensor data. 

```c++
      for (int i = 0; i < 150; i++) {
      if (myICM.dataReady()) {
        t1 = millis();
        myICM.getAGMT();
        acc_x[i] = myICM.accX();
        acc_y[i] = myICM.accY();
        acc_z[i] = myICM.accZ();
        gyr_x[i] = myICM.gyrX();
        gyr_y[i] = myICM.gyrY();
        gyr_z[i] = myICM.gyrZ();
        t2 = millis();
        time[i] = (t2-t1);
        time_sum += time[i];
      }
    }
    float time_avg = time_sum / 150;
    Serial.println(time_sum);
    time_sum = 0;
```
Next I atempted to send the ToF data with the IMU data from the Artemis to my laptop over bluetooth. To accomplish this I decided to send all the data each time it was collected in one array and just repeat that for the desired amount of time. This was the ideal way to collect data because it allowed me to keep all the data together with one time stamp and would allow me to parse it all at once more easily. It would work to separate the data into different arrays but that would just require more parsing. Having the artemis send each iteration of data at once allows the data to be sent quicker. The following code shows the implmentation:

```c++
case GET_DATA:
      {
        tx_estring_value.clear();
        for (int i = 0; i < 750; i++) {
          if (myICM.dataReady()) {
            myICM.getAGMT();
            distanceSensor1.startRanging();  // Write configuration bytes to initiate measurement
            distanceSensor2.startRanging();
            T[i] = millis();
            Dist1[i] = distanceSensor1.getDistance();
            Dist2[i] = distanceSensor2.getDistance();
            acc_x[i] = myICM.accX();
            acc_y[i] = myICM.accY();
            acc_z[i] = myICM.accZ();
            gyr_x[i] = myICM.gyrX();
            gyr_y[i] = myICM.gyrY();
            gyr_z[i] = myICM.gyrZ();
            distanceSensor1.clearInterrupt();
            distanceSensor1.stopRanging();
            distanceSensor2.clearInterrupt();
            distanceSensor2.stopRanging();
            // Serial.print("i: ");
            // Serial.println(i);
          }
        }
        // Sends ToF
        for (int k = 0; k < 750; k++) {
          tx_estring_value.clear();
          tx_estring_value.append("T:");
          tx_estring_value.append(T[k]);
          tx_estring_value.append("|");
          tx_estring_value.append("D1:");
          tx_estring_value.append(Dist1[k]);
          tx_estring_value.append("|");
          tx_estring_value.append("D2:");
          tx_estring_value.append(Dist2[k]);
          tx_estring_value.append("|");
          tx_estring_value.append("AX:");
          tx_estring_value.append(acc_x[k]);
          tx_estring_value.append("|");
          tx_estring_value.append("AY:");
          tx_estring_value.append(acc_y[k]);
          tx_estring_value.append("|");
          tx_estring_value.append("AZ:");
          tx_estring_value.append(acc_z[k]);
          tx_estring_value.append("|");
          tx_estring_value.append("GX:");
          tx_estring_value.append(gyr_x[k]);
          tx_estring_value.append("|");
          tx_estring_value.append("GY:");
          tx_estring_value.append(gyr_y[k]);
          tx_estring_value.append("|");
          tx_estring_value.append("GZ:");
          tx_estring_value.append(gyr_z[k]);
          tx_estring_value.append("|");
          tx_characteristic_string.writeValue(tx_estring_value.c_str());
        }
        Serial.println("Sent IMU & ToF Data!");
        break;
      }    
```
This is what the output looked like on my laptop:

![image](https://user-images.githubusercontent.com/123790450/222990988-de76ddcf-c6e3-41f8-ba48-c5c22aef43cd.png)

Looking at the initial and final timestamped data you can see at least five seconds have passed.

To figure out the maximum time I could record data for I looked at the output of my current program that recorded five seconds.

![image](https://user-images.githubusercontent.com/123790450/222992035-ea5d71b0-7f6d-4e66-88cb-3c56d9680efe.png)
First I calculated how much of the used space was because of the arrays I already used. I used 9 750 length float arrays. This took up a total of 27,000 bytes (9\*750\*4). The current program took up 72,520 bytes so that meant the program took up 45,520 bytes without the data arrays. With a maximum memory size of 393216 bytes, this left 347696 bytes of data left for data storage. With 9 data arrays of floats needing to be made this mean there was space for 9 float arrays with a length of 9658. In my testing an array of 150 length represented 1 seconds. This means I could take a max of 64 seconds worth of data.

## Cut the Cord

Having been given an 850 mAh battery and a 650 mAh battery, I decided to use the 650 mAh battery to power the arduino and the 850 mAh to power the RC car. This was because the motors of the car will most likely use more power than the Artemis. Additionally powering the sensors and motors off of separate power sources is a good way to reduce EMI. To make the entire process easy I decided to solder an additional connector onto the battery. This would allow me to hotswap the batteries more easily. The below image shows the Artemis powered off of the battery.

![image](https://user-images.githubusercontent.com/123790450/222992742-e56d05ae-40e5-41ac-9bb9-3da6b00e667e.png)

The Artemis was able to successfully communicate while plugged in as demonstrated by the image below:

![image](https://user-images.githubusercontent.com/123790450/222992891-f0cde59c-2238-42e8-b780-bb8bdd2fbb5b.png)

## Record a Stunt

I tested the RC car for a bit. Some of the things I noticed were it would bounce around when hitting obstacles due to it's large wheel size. I also noticed it could be run in rightside up and upside down because the wheels extended past the frame on both sides. This means that if it hit an obstacle it would just flip over. Another thing I noticed is the remote wouldn't drive the car forwards or backwards and would only allow the car to drive if it was also turning. This was likely just an issue with a controller because that would be a very limiting design choice for a toy. Here is a video of me using the car.

<iframe width="560" height="315" src="https://www.youtube.com/embed/qiYTEfz9gIU" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

It was difficult to get the robot to perform a stunt because of the broken remote control. To demonstrate the IMU data collection was working I decided to just rotate the car by hand to see the received data. Here is the IMU positioned on the robot:

![image](https://user-images.githubusercontent.com/123790450/222995307-b3890f22-25ad-4b5d-838d-f705169e1c79.png)

Here is how I rotated the car by hand:
<video src="https://user-images.githubusercontent.com/123790450/222996986-a7ebed7e-8ca4-4db1-9ec4-6795839bb734.mp4" controls="controls" style="max-width: 730px;">
</video>

After rotating the entire robot by hand in different orientations here is the output from the IMU and ToF:
![image](https://user-images.githubusercontent.com/123790450/222995843-1981707d-a949-461e-a448-0a67f5ae32a1.png)
![image](https://user-images.githubusercontent.com/123790450/222995850-7003bee7-3a65-44bb-b764-491f8bd48e76.png)
![image](https://user-images.githubusercontent.com/123790450/222995859-b55274cf-d70c-48fa-ba21-58d929a53dba.png)

I have now succesfully integrated an IMU into my robot as well as developed a method to process the signals from the sensor.


