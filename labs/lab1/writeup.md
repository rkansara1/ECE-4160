---
title: Lab 1: The Artemis Board
---

## Objective
The purpose of this lab was to get set up with the Arduino IDE and the Artemis Nano board. I worked through the following example sketches.

## Examples

### Example 1: Blink it Up

This example was very straightforward to setup. All that needs to be done is to go to Files>Examples>01.Basics>Blink. Uploading that file will have the onboard Artemis  LED blinking. The baud rate needs to be changed to 115200. The video below shows the Artemis board blinking.

<video src="https://user-images.githubusercontent.com/123790450/216876429-c1f520b4-aeba-459b-be28-132f4c248500.mp4" controls="controls" style="max-width: 730px;">
</video>

### Example 2: Serial Monitor

The next example can be found under Files>Examples>Apollo3>Example04_Serial. In this example the Artemis will write to the serial monitor a series of messages. The messages can be seen below in the image. Then the user has the ability to have the artemis print the message they write into the serial monitor. I did this with the string "Hello Artemis".

<img width="427" alt="example04serial" src="https://user-images.githubusercontent.com/123790450/216878484-48f9fd4d-0844-4f7c-8878-5a02ff9583ae.png">




### Example 3: Analog Read

This example can be found under Files>Examples>Apollo3>Example02_AnalogRead. In this example the artemis will read the internal die temperature. In the below video you can observe how the die temperature started around 79.5°F. I placed my finger on the outside of the chip and it increased steadily to around 84°F. When I stopped pressing it the temperature then drops back down to about 79.5°F. I modified the example code in order to get the program to print out the internal temperature in Fahrenheit as well as display the temperature on the serial plotter. Here are the code modifications:

```c
void loop() {
#ifdef ADCPIN
  int external = analogRead(EXTERNAL_ADC_PIN); // reads the analog voltage on the selected analog pin
  //  I commented out the below line because it was preventing the serial plot from working
  // Serial.printf("external (counts): %d, ", external);
  analogWrite(LED_BUILTIN, external);
#endif

  int vcc_3 = analogReadVCCDiv3();    // reads VCC across a 1/3 voltage divider
  int vss = analogReadVSS();          // ideally 0
  int temp_raw = analogReadTemp();    // raw ADC counts from die temperature sensor
  
  float temp_f = getTempDegF();       // computed die temperature in deg F
  float vcc_v = getVCCV();            // computed supply voltage in V

  // I commented out the line that prints out all the data and instead only print out the temp_F variable with a slight delay
  // to see the temperature progression better.
  // Serial.printf("temp (counts): %d, vcc/3 (counts): %d, vss (counts): %d, time (ms) %d\n", temp_raw, vcc_3, vss, millis());
  Serial.println(temp_f);
  delay(250);
}
```
<video src="https://user-images.githubusercontent.com/123790450/216880467-aa27edad-ca0b-442a-8de2-fe06849ff66c.mp4" controls="controls" style="max-width: 730px;">
</video>

### Example 4: Microphone Output

This example is found under Files>Examples>PDM>Example1_MicrophoneOutput. I uploaded the provided sketch to the Arduino. I attempted whistling and it brought the loudest frequency value from the 18000s to the 2000s. This can be observed in the below video. The loudest frequency is likely so low because the fans of my laptop are the only ambient noise.



<video src="https://user-images.githubusercontent.com/123790450/216882377-1052b358-2988-4db1-8d40-455185568920.mp4" controls="controls" style="max-width: 730px;">
</video>


