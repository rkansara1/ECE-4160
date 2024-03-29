---
title: "Lab 9: Mapping"
layout: page
---

### Goal

In this lab I was tasked with mapping out a static room. This static room will be used in future labs, so obtaining a quality mapping of it will help with future efforts.

![room](room.jpg)

### Control

To map out the room, my robot needs to be able to rotate at either a constant rate or to constant positions. I chose to design an closed loop controller for the former. To measure the angle of rotation, in this case yaw, I needed to use the IMU. Luckily in [Lab 4](/ECE-4160/labs/lab4/writeup), I developed a method to use the IMU and get the yaw of the car.

The first initial problem with performing this type of control was sensor drift. Over time, the yaw angle slowly drifts even if the car remains still. This means given enough time the robot will believe it is at 180 degrees even if it hasn't moved. To fix this issue, I sampled the IMU over set time period, observed the drift, and then subtracted that value from the sensor output. This way the sensor drift would be accounted for. Here is the code I used to achieve this:

```c++
case CALIBRATE_IMU:
    {
    float yaw_g_calibrate = 0;
    float t0 = millis();
    float tNow = t0;
    float deltaT = 0;
    for (int inc = 0; inc < 10000; inc++) {
        while (!myICM.dataReady()) {
        delay(1);
        }
        myICM.getAGMT();
        deltaT = (millis() - tNow) / 1000.0;
        tNow = millis();
        yaw_g_calibrate = yaw_g_calibrate + myICM.gyrZ() * deltaT;
        Serial.println(yaw_g_calibrate);
    }
    IMU_Drift_Constant = yaw_g_calibrate / ((millis() - t0) / 1000.0);
    Serial.println(yaw_g_calibrate);
    Serial.println(millis() - t0);
    Serial.println(IMU_Drift_Constant);
    tx_estring_value.clear();
    tx_estring_value.append("IMU_Drift_Consant: ");
    tx_estring_value.append(IMU_Drift_Constant);
    tx_characteristic_string.writeValue(tx_estring_value.c_str());
    }
```

Running this block of code resulted in 11.15 degrees of drift over 40594 ms, giving a drift constant of 0.27. I then incorporated this to where I was calculating yaw.

`yaw_g = yaw_g + myICM.gyrZ() * dt + (IMU_Drift_Constant * dt);`

Implementing the rest of the control loop was simple because it reused the same principals developed in [Lab 6](/ECE-4160/labs/lab6/writeup). Here is what the control loop looks like:

<script src="https://gist.github.com/rkansara1/739f17305bda2feb67c97bfa1b7294ff.js"></script>

The robot rotates at 10 degree increments. After it reaches within 2.5 degrees of each ten degree increment it samples the ToF sensor 5 times into an array. After going through a full 360 degree rotation, the data is sent back to the computer.

Here is a video of the robot successfully rotating:

<iframe width="560" height="315" src="https://www.youtube.com/embed/vUQCTslon9o" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

### Mapping
Next I mapped out the room with the given coordinates. Here are the measuring locations.
![image](measurement.png)

At each point I ran the orientation control loop. Here are the results of mapping:

|              | Polar             | Cartesian             |
|--------------|-------------------|-----------------------|
| Origin (0,0) | ![](P0_Polar.png) | ![](P0_Cartesian.png) |
| P1 (-3,-2)   | ![](P1_Polar.png) | ![](P1_Cartesian.png) |
| P2 (0,3)     | ![](P2_Polar.png) | ![](P2_Cartesian.png) |
| P3 (5,3)     | ![](P3_Polar.png) | ![](P3_Cartesian.png) |
| P4 (5,-3)    | ![](P4_Polar.png) | ![](P4_Cartesian.png) |

To translate from polar coordinates I just used a simple polar to cartesian relation. 
$$x = r * cos(\theta)$$
$$y=r*sin(\theta)$$

Here is my processing code:
```python
with open ("origin_Dist.pkl",'rb') as file:
    Dist_Origin = pickle.load(file)
with open ("origin_Yaw.pkl",'rb') as file:
    Theta_Origin = pickle.load(file)
Dist_Origin = np.array(Dist_Origin[0:185])
Theta_Origin = np.array(Theta_Origin[0:185])
Dist_Origin = np.array(Dist_Origin)/304 # Convert from mm to ft
Theta_Origin = np.deg2rad(Theta_Origin) # Convert from degrees to radians
```


Using a more complicated rotation matrix was not necessary because the robot starts at each position with the exact same orientation.

I then combined the outputs to receive this mapping:

![](combined_map.png)

Using the map I overlaid lines over it to signify the walls. I slightly corrected curves and diagonal lines because I know the walls are not crooked and are orthogonal with each other. I also trusted the points from the measuring location that was closest. For example for the lower left corner, I trusted the blue P1 points more than the red origin points or the green P2 points because the P1 points were taken from a closer position.

![](map_walls.png)

There are a few sources of error for this map overall:

1) The orientation control was imprecise and didn't always lead to the exact angle. The robot drives with skid steering, because of friction with the floor the wheels aren't able to turn perfectly so the robot actually goes in a circle rather than rotating in place.

2) The control law is a simple P controller. Due to time constraints I did not add an integrator or derivative term to my control law. This may not have made a huge difference because the car always was able to get within 2.5 degrees of the reference

3) The robot was placed within the square tile but wasn't precisely placed at an exact coordinate. I placed the robot on the sticker that marked the location but it may have been more precise to line up a corner of the robot with a corner of a tile.

4) I should have used both ToF sensors to obtain a higher quality map. This would give two different sides of the rotation. Or I could have placed both ToF sensors in the front to get more reliable distance values.

5) I could have incremented at smaller reference values. At the moment the robot goes [0,10,20,...,360], I could have done [0,5,10,...360]. The tradeoff here is the time it takes to map. More reference positions means more time to take measurements. Additionally I didn't have a high trust in the fidelity of the angle so I knew there was a limitation to the angle to which I could go to.