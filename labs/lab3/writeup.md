---
title: "Lab 3: Time of Flight Sensors"
---

Date: 2/17/22

## Prelab

My robot will have two Time of Flight Sensors on it. To allow the use of two time of flight sensors with the same I2C address one of the sensors just needs to have its I2C address changed from the default of Ox52. To do this I will take advantage of the `XSHUT` pin on the sensor. If the `XSHUT` pin is set to low on of the ToF sensors, then it is possible to change the I2C address programatically on the other sensor and then turn the `XSHUT` pin back to high to have two working ToF sensors.

In order to ensure I have the best view possible I plan on putting one ToF sensor on the front of the robot and one on the side of the robot (as seen in the below image). This way I should be able to tell what is in front of me and what is to the side of me. Additionally by rotation 180 degrees I can capture a full 360 degree field of obstacles. A scenario I can miss an object is in the beginning with where my robot gets placed. If an obstacle is in the blindspot of the robot then I won't be able to see it initially and might not be able to make the full 180 degree turn to capture obstacles.

<img width="515" alt="image" src="https://user-images.githubusercontent.com/123790450/219848689-880f7746-33de-4246-abc5-9a311a87238b.png">

Wiring Diagram:

![image](https://user-images.githubusercontent.com/123790450/219849323-3d00c0a3-a3c9-45fd-9bfc-aea1e813e9e7.png)
Note: `XSHUT` is connected to `PIN 8`


## Lab Tasks

ToF Sensor Connected to QWIIC Breakout Board:
![IMG_1007](https://user-images.githubusercontent.com/123790450/219849507-41d61cb9-59ab-4d6d-ab12-1914e69c0c50.jpg)

Artemis Scanning for I2C Device:

![image](https://user-images.githubusercontent.com/123790450/219849611-c7c522bd-9918-4017-9ac8-4b94a4bf77ba.png)
Something interesting occurred while scanning for I2C devices. The address show is Ox29 instead of the expected Ox25. The reason for this is that 0x29 is just 0x52 with one bit used to know if the devices is read or writing instead of for the address. 0x52 in binary is 0b1010010 while 0x29 0b101001. 0x29 is just 0x52 with a right bit shift.

Sensor Mode:
The ToF sensors have three different distance modes that can be chosen from. They are:
![image](https://user-images.githubusercontent.com/123790450/219849870-7929639e-1f39-45f3-ac8c-040232382499.png)

A function in the included library allows to easily switch between modes. The medium distance mode is not able to be set from the library however. There is only a `setDistanceModeShort()` and `setDistanceModeLong()`. After performing a test by placing a piece of paper and seeing how accurate the sensor was in either mode, I was able to conclude that the long mode suited my situation better. Additionally in terms of the robots performance being able to see the furthest out will help the most with planning routes to take rather than just having higher fidelity nearby distances.

Using 11.5" benchmark:

![image](https://user-images.githubusercontent.com/123790450/219849972-ff945a06-76f8-47b4-8cf2-db638537452b.png)

Distance Mode Long:

![distanceModeLong](https://user-images.githubusercontent.com/123790450/219849984-6591ce80-b2c2-4cb7-b912-ed027df44211.png)

Distance Mode Short:

![distanceModeShort](https://user-images.githubusercontent.com/123790450/219849987-8206030b-b8b8-4687-b769-8242588e42b1.png)


### 2 ToF Sensors
Using the strategy discussed in Prelab it was very straightforward to get two ToF sensors working together. In the void setup all that needed to be included was:
```c++
 digitalWrite(SHUTDOWN_PIN, LOW);
 distanceSensor1.setI2CAddress(10);
 digitalWrite(SHUTDOWN_PIN, HIGH);
```
Two ToF Sensors Connected:

![image](https://user-images.githubusercontent.com/123790450/219850154-4e04aa97-cb9f-4290-92c3-c708930cc545.png)

Output (One sensor is facing a wall the other is pointed at my ceiling):

![image](https://user-images.githubusercontent.com/123790450/219850270-74c14b81-7971-4165-a4a1-ae879a90eecc.png)

To benchmark the limiting factor and seeing how fast I could take reading from the ToF sensors I wrote a simple loop that printed out data as soon as it came in.

Arduino Code:

```c++
  distanceSensor1.startRanging();
  distanceSensor2.startRanging();
  if (distanceSensor1.checkForDataReady()) {
    int distance1 = distanceSensor1.getDistance();
    distanceSensor1.clearInterrupt();
    distanceSensor1.stopRanging();
    Serial.print("Distance 1 (mm): ");
    Serial.print(distance1);
    Serial.print("   Time (ms): ");
    Serial.print(millis());
  }
  if (distanceSensor2.checkForDataReady()) {
    int distance2 = distanceSensor1.getDistance();
    distanceSensor2.clearInterrupt();
    distanceSensor2.stopRanging();
    Serial.print("Distance 2 (mm): ");
    Serial.print(distance2);
    Serial.print("   Time (ms): ");
    Serial.print(millis());
  } else {
    Serial.print("   Time (ms): ");
    Serial.println(millis());
  }
```
Output:

![image](https://user-images.githubusercontent.com/123790450/219850378-0bb81594-cf88-44eb-a6f7-15fb02aaea0f.png)

Analyzing this reveals that it takes about 80ms for the system to send two ToF sensor readings out. Based on how fast this output is I should be able to use my sensors to accurately track distances.


### Graphing Data
The next step was to now send this data to my laptop in order to analyze the data without having to be wired in. I reused the notifaciton handler and the robot command code from the previous lab. I only had to add in a new command that took in the ToF sensors (as well as integrating all the setup with the I2C and libraries) and do some simple data processing to generate graphs.

Arduino Code:
```c++
case GET_TOF_DATA:
      {
        int initial_t = millis();
        tx_estring_value.clear();
        int time_now = initial_t;
        for (int i = 0; i < 150; i++) {
          distanceSensor1.startRanging();  // Write configuration bytes to initiate measurement
          distanceSensor2.startRanging();
          T[i] = millis();
          Dist1[i] = distanceSensor1.getDistance();
          Dist2[i] = distanceSensor2.getDistance();
          distanceSensor1.clearInterrupt();
          distanceSensor1.stopRanging();
          distanceSensor2.clearInterrupt();
          distanceSensor2.stopRanging();
        }
        for (int k = 0; k < 145; k += 5) {
          tx_estring_value.clear();
          for (int i = k; i < k + 5; i++) {
            tx_estring_value.append("T:");
            tx_estring_value.append(T[i]);
            tx_estring_value.append("|");
            tx_estring_value.append("D1:");
            tx_estring_value.append(Dist1[i]);
            tx_estring_value.append("|");
            tx_estring_value.append("D2:");
            tx_estring_value.append(Dist2[i]);
            tx_estring_value.append("|");
          }
          tx_characteristic_string.writeValue(tx_estring_value.c_str());
        }
        Serial.println("Sent Data!");
        break;
      }
```

Python Code:

```python
s = []
async def string_notification(uuid, bytearray):
    s.append(ble.bytearray_to_string(bytearray))
        
ble.start_notify(ble.uuid["RX_STRING"],string_notification)
ble.send_command(CMD.GET_TOF_DATA,"")
T = []
D1 = []
D2 = []
for i in range(len(s)):
    batch = s[i].split("|")
    for j in range(len(batch)-1):
        data = batch[j]
        if data[0] == "T":
            T.append(float(data[3:]) )
        if data[:2] == "D1":
            D1.append(float(data[3:]) )
        if data[:2] == "D2":
            D2.append(float(data[3:]) )

T_sec = np.array(T)
T_sec = T_sec/1000
plt.plot(T_sec,D1,label = 'Sensor 1')
plt.plot(T_sec,D2, label = 'Sensor 2')
plt.legend()
```

Graph:

![image](https://user-images.githubusercontent.com/123790450/219850579-e6b886c8-72dd-4a8a-936d-8d344db6fdd9.png)

With all of this done I should be able to accurately receive ToF data from my Arduino without being plugged in.





