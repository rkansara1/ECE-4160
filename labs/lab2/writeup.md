---
title: "Lab 2: Bluetooth"
---

Date: 2/14/22

## Prelab

### Computer Setup
Setting up my computer for the lab was an incredibly difficult process. I am running a Windows 11 PC. Due to an unknown bug I am unable to connect to bluetooth using Windows 11. What I instead did was installed the latest version of [Ubuntu](https://ubuntu.com/download/desktop) and set up a dual boot for my PC. After following the provided setup process for installing Python and the starter code my computer was able to connect with the Artemis Nano.

To initially connect to the Artemis I first had it print out its MAC adress. This was because the class has a batch of Artemis Boards which can have repeating MAC adresses. Additiionally I had the Jupyter Notebook print out a unique UUID to allow the computer and the Artemis to connect. Without a unique UUID I would have been able to accidentally connect to someone elses Artemis. 

![image](https://user-images.githubusercontent.com/123790450/218936223-58eb0ee1-40bf-448c-a938-64925845a9c7.png)

### How the codebase and Bluetooth work
The provided codebase provides a few helpful functions. The Jupyter Notebook has a BLEService which provides functions for connecting the notebook to the Artemis. The function `ble.connect()` allows the notebook to connect to the Artemis. Here is what that looks like:

![image](https://user-images.githubusercontent.com/123790450/218937721-ca65f610-3b91-4eaf-8e0f-34b0a2916a9f.png)

![image](https://user-images.githubusercontent.com/123790450/218937771-e4c33b48-0364-43ed-ac14-72c17414c6fe.png)

Bluetooth LE works with the Artemis being the peripheral device and the laptop as the central device. The Artemis is like a bulletin board with the laptop then reading that board. The information is structured into services and then into characteristic. A service is like a notice on the bulletin board and the characteristic is like a parapgraph on that notice. A service is identified with a UUID, so think of each notice on a bulletin board having it's own name, and then the characteristic is 512 bytes long on that service. In this lab we use two services for `RX_FLOAT` and `RX_STRING`, is The Bluetooth LE specification has a feature called notify. When notify is enabled on a characteristic and the peripheral writes new data on the characteristic the central device is automatically sent that data. This will allow us to create a communciation interface with our Artemis and our laptops to send ToF data, IMU data, etc.

## Lab Tasks

### Configurations
Here are the necessary configurations which needed to be made:
Updating the `connections.yaml` file with the Artemis's MAC Address as well as changing the `ble_service` UUID with a newly generated one.

![image](https://user-images.githubusercontent.com/123790450/218940423-10ee21c8-04ae-4600-a2b1-5cb922eddbf5.png)

Updating the `ble_arduino.ino` file with same ble_service UUID

![image](https://user-images.githubusercontent.com/123790450/218940691-e28c5ad3-8d8d-4ff5-85f1-8a25528a56db.png)

### demo.ipynb
Prior to adding the notificaiton handler the `demo.ipynb` file contained a few given functions such as `PING` and `SEND_TWO_INTS`. Before carrying out all lab tasks here is what the Jupyter Notebook and Arduino Serial Monitor display. (`ble.disconnect()` was ran in a lower cell but not inlcuded in the screenshot)

![image](https://user-images.githubusercontent.com/123790450/218941576-9f667fa8-684c-4d6e-a340-4b7508caff24.png)

![image](https://user-images.githubusercontent.com/123790450/218941636-b0fbe44c-742d-40d4-acd7-381213b1dd1d.png)

### Send Echo Command
The `ECHO` command just takes an input string from the laptop, sends it to the Artemis, displays it to the Artemis's serial monitor, augments the message and sends it back to the laptop to be read. Below is the output on both the Jupyter Notebook and the serial monitor. The command takes advantage of `ble.send_command()` function as well as the `ble.receive_string()` function.

![image](https://user-images.githubusercontent.com/123790450/218943983-a77e5d49-206b-4e70-af11-13bd99165bdd.png)


Python Code:

<script src="https://gist.github.com/rkansara1/2c24fb483f15776e12e0e667a895b8ed.js"></script>

Arduino Code:

<script src="https://gist.github.com/rkansara1/8a7b11c517f778abaa272e893287456f.js"></script>


### Get Time Command
To complete the `GET_TIME_MILLIS` command, a new enum and robot command needed to be added to the list. The `cmd_types.py` file had to be updated with a new enum called `GET_TIME_MILLIS`. The `ble_arduino.ino` CommandTypes needed a new enum added to it with the same name. Additionally another case needed to be added to the switch statement in the same file. The `GET_TIME_MILLIS` command works by sending a command to the arduino. The arduino then takes the time it's been running with the `millis()` function. Then it sends that string back to the laptop to be printed in the Jupyter Notebook. Below is the code snippets as well as the output seen.

Output:

![image](https://user-images.githubusercontent.com/123790450/218945664-1c9f9880-930b-43ab-9f11-9f9abb95bfc9.png)

Python Code:

```python
ble.send_command(CMD.GET_TIME_MILLIS,"")
time_millis = ble.receive_string(ble.uuid['RX_STRING'])
LOG.info(time_millis)
```
Arduino Code:

```c++
case GET_TIME_MILLIS:
      {
        int time_millis = millis();
        tx_estring_value.clear();
        tx_estring_value.append("T:");
        tx_estring_value.append(time_millis);
        tx_characteristic_string.writeValue(tx_estring_value.c_str());
        break;
      }
```

### Notification Handler

The notification handler takes advantage of the notify feature of the Bluetooth Low Energy specification. To enable it a callback function needs to be created. Here is the created callback function: 

```python
async def string_notification(uuid, bytearray):
    s = ble.bytearray_to_string(bytearray)
    LOG.info(f"{s}")
```
After that
```python
ble.start_notify(ble.uuid["RX_STRING"],string_notification)
```
is called. This starts the notification process. Now if `ble.send_command(CMD.GET_TIME_MILLIS,"")` is run it will print out the string that is received without a print command needed. Below is an example of this output.

![image](https://user-images.githubusercontent.com/123790450/218947615-dbf7201f-5b78-4ddf-b61f-662bfeb86eb0.png)

### Get Temperature Command

Next, the `GET_TEMP_5s` and `GET_TEMP_5s_RAPID` commands were created. A similar process was followed as with adding the `GET_TIME_MILLIS` command in adding the right enums and modifying the switch statement. To print out the temperature a built in function called `getTempDegC()` was used to return the die temperature in Celsius. To have the Artemis print out the temperature every second for five seconds a for loop was used that appended the timestamped temperature to the output string, waited 1 second and then repeated. Below is the code and the output.

Arduino Code:
```c++
case GET_TEMP_5s:
      {
        //make this into a string
        tx_estring_value.clear();
        for (int i = 0; i < 5; i++) {
          int time_millis = millis();
          float temp = getTempDegC();
          tx_estring_value.append("T:");
          tx_estring_value.append(time_millis);
          tx_estring_value.append("|");
          tx_estring_value.append("C:");
          tx_estring_value.append(temp);
          tx_estring_value.append("|");
          delay(1000);
        }
        tx_characteristic_string.writeValue(tx_estring_value.c_str());
        break;
      }
```
Python Code:
```
ble.send_command(CMD.GET_TEMP_5s,"")
```
Output:

![image](https://user-images.githubusercontent.com/123790450/218948668-ca268fa1-d343-4399-8f14-afb4580cff70.png)

Next to implement `GET_TEMP_5s_RAPID`, I created a while loop that just checked if the current time - start time was less than 5000 ms. If it wasn't the program sent out the timestamped die temperature.

Arduino Code:
```c++
 case GET_TEMP_5s_RAPID:
      {
        int initial_t = millis();
        tx_estring_value.clear();
        int time_now = initial_t;
        while ((time_now - initial_t) < 5000) {
          tx_estring_value.clear();
          float temp = getTempDegC();
          tx_estring_value.append("T:");
          tx_estring_value.append((int)millis());
          tx_estring_value.append("|");
          tx_estring_value.append("C:");
          tx_estring_value.append(temp);
          tx_estring_value.append("|");
          tx_characteristic_string.writeValue(tx_estring_value.c_str());
          time_now = millis();
        }
        break;
      }
```

Python Code:

```python
ble.send_command(CMD.GET_TEMP_5s_RAPID,"")
```

Output: (The image is small so it will help to open it a new tab)

![image](https://user-images.githubusercontent.com/123790450/218949873-468f5669-73ad-4996-b850-8a056f772b9b.png)


### Limitations

5 seconds of 16 bit values taken at 150 Hz is 12,000 bits or 1.5 KB. The Artemis has a 384 KB of RAM and the program used takes up almos 30 KB. This means there is about 350 KB left for variables. This means the Artemis can store 233 1.5 KB sets of data.
