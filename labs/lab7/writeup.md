---
title: "Lab 7: Kalman Filter"
---

## Estimating Drag and Momentum

The first step in building a kalman filter is to determine the state space matrices that represesnt the physical process occurring. In this case we need to estimate the drag and momentum terms in order to get an accurate model representation. First, to estimate these two terms I performed a step response and observed the results. In this case I set my pwm to 100 and had the robot crash into the wall and then I collected the distance data. Using the distance data I was able to figure out the velocity during the step response because the velocity is just the time it took inbetween each distance array value. I found that the constant velcoity was approximately 2300 mm/s, as can be seen below. Next I used the given equation in class to find my "mass" value of 4.25E-4. 


![image](https://user-images.githubusercontent.com/123790450/228410750-0de1a454-4d8e-4a01-9336-769d0f753831.png)

![image](https://user-images.githubusercontent.com/123790450/228410770-16e0962c-8d66-44ad-9628-d43cda556326.png)

Equations used:
![image](https://user-images.githubusercontent.com/123790450/228410946-5a6f8e27-c7fa-4a4c-a542-ba0b901c8882.png)

![image](https://user-images.githubusercontent.com/123790450/228410979-93c5cd06-c87e-463a-876d-3a9e3e342104.png)



## Initializing KF

## Sanity Check

## Extrapolation
