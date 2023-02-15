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
Prior to adding the notificaiton handler the `demo.ipynb` file contained a few given functions such as `PING` and `SEND_TWO_INTS`. Before carrying out all lab tasks here is what the Jupyter Notebook and Arduino Serial Monitor display. (`ble.disconnect` was ran in a lower cell but not inlcuded in the screenshot)

![image](https://user-images.githubusercontent.com/123790450/218941576-9f667fa8-684c-4d6e-a340-4b7508caff24.png)
![image](https://user-images.githubusercontent.com/123790450/218941636-b0fbe44c-742d-40d4-acd7-381213b1dd1d.png)


