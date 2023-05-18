---
title: "Lab 11: Localization on the Real Robot "
layout: page
---

# Objective
The objective of this lab is to perform localization on the robot with the Bayes filter implemented.

# Simulation
First, I ran through the simulation file to see a comparison between the odometry model, the bayes filter model, and the actual position.

Screenshot of the result:

![sim](sim.png)

The green represents the ground truth, the blue represents the bayes filter, and the red represents the odometry model. The bayes filter is able to more closely track the correct position of the car throughout its trajectory. The odometry model very quickly goes off track.


# Localization

My first step was to implement `perform_observation_loop` in the `RealRobot` class. 
Here is my implementation:

<script src="https://gist.github.com/rkansara1/37ec51c0559462bf4c8de3d9309b3b18.js"></script>

Additionally I needed to add a notification handler:

```python
s = []
async def string_notification(uuid, bytearray):
    s.append(ble.bytearray_to_string(bytearray))
    LOG.info(ble.bytearray_to_string(bytearray))
        
ble.start_notify(ble.uuid["RX_STRING"],string_notification)
```

`perform_observation_loop` is implemented similarly to [Lab 9](/ECE-4160/labs/lab9/writeup.md). One change I needed to make was have the program pause while data was still being collected. To do this I just check the length of the string list coming in from the notification handler. Once it has collected 18 strings it knows the data collection is over and it can now process the values. The program takes 5 distance measurements at each angle increment to account for any poor sensing, so in the function each grouping of 5 is averaged.


# Localization Results

Here are the results of running the localization at the four marked points in lab. The points in lab are measured in feet but everything is converted into meters for the simulation. ![](5,-3 round 2.png)


|            | Simulation        | Polar                 | Belief            |
|------------|-------------------|-----------------------|-------------------|
| P1 (-3,-2) | ![](P1_Polar.png) | ![](P1_Cartesian.png) |                   |
| P2 (0,3)   | ![](P2_Polar.png) | ![](P2_Cartesian.png) |                   |
| P3 (5,3)   | ![](P3_Sim.png)   | ![](P3_Polar.png)     | (1.829,0.610,10)  |
| P4 (5,-3)  | ![](P4_Sim.png)   | ![](P4_Polar.png)     | (1.829,-1.219,10) |