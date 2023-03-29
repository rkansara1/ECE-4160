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

My next step was initializing the Kalman Filter. The below code shows the initializaiton of the filter.

``` python
d = 1/2300
m = 0.000425
A = np.array([[0,1],[0,-d/m]])
B = np.array([[0],[1/m]])
C = np.array([[1,0]])

X = np.array([[D2[0]],[V[0]]])
sigma = np.array([[10,0],[0,0.1]])

sigma_1 = 20
sigma_2 = 20
sigma_3 = 5

sig_u=np.array([[sigma_1**2,0],[0,sigma_2**2]])
sig_z=np.array([[sigma_3**2]])
```
My C matrix is different in this case because I am measuring my output as a positive distance from the wall which is why it is \[1 0] compared to the \[-1 0] seen in class. I initially set my sigma_1 and sigma_2 relatively larger than my sigma_3 because I had more trust in my measurement then my model. Additionally I set my initial sigma matrix to have some confidence in the distance measurement and incredibly high confidence in the initial velocity state. This was because my robot was starting from rest so it almost certainly has an inital velocity of zero.

## Sanity Check
Next I discretized my matrix by averaging my sampling time from my distance matrix, approximately 0.1 seconds, and used the provided kalman filter code to implement my kalman filter.

``` python
Ad = np.eye(2) + A*dt
Bd = B * dt

def kf(mu,sigma,u,y):
    
    mu_p = Ad.dot(mu) + Bd.dot(u) 
    sigma_p = Ad.dot(sigma.dot(Ad.transpose())) + sig_u
    
    sigma_m = C.dot(sigma_p.dot(C.transpose())) + sig_z
    kkf_gain = sigma_p.dot(C.transpose().dot(np.linalg.inv(sigma_m)))

    y_m = y-C.dot(mu_p)
    mu = mu_p + kkf_gain.dot(y_m)    
    sigma=(np.eye(2)-kkf_gain.dot(C)).dot(sigma_p)

    return mu,sigma

kf_state = []
kf_state.append(D2[0])

for i in range(len(D2)-1):
    X, sigma = kf(X,sigma,Input[i]/100,D2[i])
    kf_state.append(X[0,:])

```
My Input has to be divided by 100 because my step size was actually 100 pwm. Next using values from a PID test from the previous lab I was able to see how my kalman filter worked.

![image](https://user-images.githubusercontent.com/123790450/228415118-ae2ab4db-e403-44f2-87b6-63b3896c294a.png).

I also tested the kalman filter with similar covariance matrices and different covariance matrices.

![image](https://user-images.githubusercontent.com/123790450/228415218-5a3709a1-70df-4c3b-b92f-b54af6635443.png)

![image](https://user-images.githubusercontent.com/123790450/228415289-19301833-6af9-4435-af92-e20f80381715.png)

These different cases show how adjusting the covariance matrix can change the way the kalman filter works. This makes sense because the covariance matrices represent how much trust you have in either you dynamic model or your sensor measurement. In this case having a filter with a lower measurement covariance matrix ended up having the filter track the data better. Having similar covaraince matrices also gave a kalman filte that tracked the value of the sensor well. Finally, having a filter that didn't trust the measurement value led to a kalman filter that went the opposite way of the initial sensor data. However after some time it began to folow the data.




## Extrapolation
