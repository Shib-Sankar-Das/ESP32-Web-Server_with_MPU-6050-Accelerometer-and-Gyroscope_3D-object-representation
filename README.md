# ESP32 Web Server with MPU-6050 Accelerometer and Gyroscope (3D object representation)
In this project we’ll build a web server with the ESP32 to display readings from the MPU-6050 accelerometer and gyroscope sensor. We’ll also create a 3D representation of the sensor orientation on the web browser. The readings are updated automatically using Server-Sent Events and the 3D representation is handled using a JavaScript library called three.js. The ESP32 board will be programmed using the Arduino core.
![ESP32-Web-Server-MPU-6050-Accelerometer-Gyroscope-3D-object-Arduino](https://github.com/user-attachments/assets/5996f7c2-79a2-4c6c-8052-d2d247c04b04)
To build the web server we’ll use the ESPAsyncWebServer library that provides an easy way to build an asynchronous web server and handle Server-Sent Events.

## Project Overview
Before going straight to the project, it’s important to outline what our web server will do, so that it is easier to understand.

![ESP-MPU6050-Web-Server-Arduino](https://github.com/user-attachments/assets/c3d27ff8-24c5-442f-b47d-1c99d0f69867)

- **Gyroscope Readings**
  - Displays **X, Y, and Z axis** values.
  - Values are updated **every 10 milliseconds**.

- **Accelerometer Readings**
  - Displays **X, Y, and Z axis** values.
  - Values are updated **every 200 milliseconds**.

- **Temperature Readings**
  - The **MPU-6050 sensor module** also measures temperature.
  - Temperature value is updated **every second (1000 milliseconds)**.

- **Data Transmission**
  - All readings are updated using **Server-Sent Events (SSE)** for real-time updates.

- **3D Sensor Representation**
  - The web server includes a **3D visualization** of the sensor.
  - The **3D object** changes orientation based on the **gyroscope values**.
  - The 3D object is created using the **Three.js JavaScript library**.

- **Control Buttons**
  - There are **four buttons** to adjust the **3D object's position**:
    - **RESET POSITION**: Sets angular position to **zero** on all axes.
    - **X**: Resets the **X-axis** angular position.
    - **Y**: Resets the **Y-axis** angular position.
    - **Z**: Resets the **Z-axis** angular position.
 
![MPU-6050-How-It-Works-ESP32](https://github.com/user-attachments/assets/b0c43754-7d2b-40c2-ba26-6c3fcb7925bf)


## ESP32 Filesystem
To keep our project organized and make it easier to understand, we’ll create four different files to build the web server:
![MPU6050-Web-Server-Project-Folder-Structure-SPIFFS](https://github.com/user-attachments/assets/49e539c9-1cd0-4053-aa83-98c401e40696)

- **Arduino Code (ESP32 Web Server)**
  - Handles the web server functionality.
  - Manages the sensor readings from the MPU-6050.
  - Serves the HTML, CSS, and JavaScript files to the browser.
  - Sends sensor data to the client using **Server-Sent Events (SSE)**.

- **HTML File**
  - Defines the **content** of the web page.
  - Contains the structure for displaying sensor readings and 3D visualization.

- **CSS File**
  - Used to **style** the web page for a better user interface.
  - Adjusts the appearance of the sensor data display, buttons, and 3D object.

- **JavaScript File**
  - Programs the **behavior** of the web page.
  - Handles **server responses** and updates the displayed sensor data.
  - Manages **events** like button clicks (resetting position).
  - Creates and controls the **3D object** using the **Three.js library**.

- **Uploading Files to the ESP32 Filesystem**
  - The HTML, CSS, and JavaScript files need to be uploaded to the **ESP32 LittleFS filesystem**.
  
  ### Uploading Files with Arduino IDE:
  - Use the **LittleFS Uploader Plugin** to upload files.
  - Follow the steps in this guide:
    - [Install ESP32 LittleFS Uploader (Upload Files to the Filesystem)](https://github.com/lorol/ESP32FS-UI)
  
  ### Uploading Files with PlatformIO + VS Code:
  - If you’re using **PlatformIO** with **VS Code**, refer to the following tutorial:
    - [ESP32 with VS Code and PlatformIO: Upload Files to LittleFS Filesystem](https://docs.platformio.org/en/latest/boards/espressif32/esp32dev.html)

## MPU-6050 Gyroscope and Accelerometer
The MPU-6050 is a module with a 3-axis accelerometer and a 3-axis gyroscope.
![MPU6050-Module-Accelerometer-Gyroscope-Temperature-Sensor (1)](https://github.com/user-attachments/assets/896cac5b-88a6-42b9-92b4-00e32e0dda34)

The gyroscope measures rotational velocity (rad/s) – this is the change of the angular position over time along the X, Y and Z axis (roll, pitch and yaw). This allows us to determine the orientation of an object.

![roll-pitch-yaw](https://github.com/user-attachments/assets/d4f36750-f712-46bd-9e10-3babc020cc09)

The accelerometer measures acceleration (rate of change of the velocity of an object). It senses static foces like gravity (9.8m/s2) or dynamic forces like vibrations or movement. The MPU-6050 measures acceleration over the X, Y an Z axis. Ideally, in a static object the acceleration over the Z axis is equal to the gravitational force, and it should be zero on the X and Y axis.

Using the values from the accelerometer, it is possible to calculate the roll and pitch angles using trigonometry, but it is not possible to calculate the yaw.

We can combine the information from both sensors to get accurate information about the sensor orientation.

## Schematic Diagram – ESP32 with MPU-6050
For this project you need the following parts:

1. **MPU-6050 Accelerometer Gyroscope**
2. **ESP32**
3. **Breadboard**
4. **Jumper wires**

Wire the ESP32 to the MPU-6050 sensor as shown in the following schematic diagram: connect the SCL pin to **GPIO 22** and the SDA pin to **GPIO 21**.

![MPU6050_ESP32_Wiring-Schematic-Diagram](https://github.com/user-attachments/assets/bb276a76-1114-4b01-b0bd-ae5bb108a851)


## Preparing Arduino IDE
We’ll program the ESP32 board using Arduino IDE. So, make sure you have the ESP32 add-on installed.

### Installing Libraries
There are different ways to get readings from the sensor. In this tutorial, we’ll use the [Adafruit MPU6050 library](https://github.com/adafruit/Adafruit_MPU6050). To use this library you also need to install the [Adafruit Unified Sensor library](https://github.com/adafruit/Adafruit_Sensor) and the [Adafruit Bus IO Library](https://github.com/adafruit/Adafruit_BusIO).

To build the web server, we’ll use the [ESPAsyncWebServer](https://github.com/ESP32Async/ESPAsyncWebServer) and the [AsyncTCP](https://github.com/ESP32Async/AsyncTCP) libraries. In this example, we’ll send the sensor readings to the browser in JSON format. To make it easier to handle JSON variables, we’ll use the [Arduino_JSON](https://github.com/arduino-libraries/Arduino_JSON) library by Arduino.

Here’s the list of libraries you need to install:

1. [**Adafruit MPU6050 library**](https://github.com/adafruit/Adafruit_MPU6050)
2. [**Adafruit Unified Sensor library**](https://github.com/adafruit/Adafruit_Sensor)
3. [**Adafruit Bus IO Library**](https://github.com/adafruit/Adafruit_BusIO)
4. [**ESPAsyncWebServer by ESP32Async**](https://github.com/ESP32Async/ESPAsyncWebServer)
5. **[AsyncTCP](https://github.com/ESP32Async/AsyncTCP) by ESP32Async**
6. **[Arduino_JSON library](https://github.com/arduino-libraries/Arduino_JSON) by Arduino**

Open your Arduino IDE and go to **Sketch > Include Library > Manage Libraries**. The Library Manager should open. Search for the libraries’ names and install them.

## Uploading Code and Files
After inserting your network credentials, save the code. Go to Sketch > Show Sketch Folder, and create a folder called data.

![arduino-ide-show-sketch-folder](https://github.com/user-attachments/assets/fb20a52a-d161-4f5b-83be-f3ab6e201155)

**Inside that folder you should save the HTML, CSS and JavaScript files.**

Then, upload the code to your ESP32 board. Make sure you have the right board and COM port selected. Also, make sure you’ve added your networks credentials to the code.

![arduino-ide-2-upload-button (1)](https://github.com/user-attachments/assets/f731f76f-3625-44c2-a280-3aa8dfe79114)

After uploading the code, you need to upload the files. In the Arduino IDE, press **[Ctrl] + [Shift] + [P]** on Windows or **[⌘] + [Shift] + [P]** on MacOS to open the command palette. Search for the **Upload LittleFS to Pico/ESP8266/ESP32** command and click on it.

![upload-files-little-fs-esp32-arduino-ide (1)](https://github.com/user-attachments/assets/e1908e9e-cdda-44da-ba86-d1b9ef094de0)

If you don’t have this option is because you didn’t install the filesystem uploader plugin.

**Important:** *make sure the Serial Monitor is closed before uploading to the filesystem. Otherwise, the upload will fail.*
When everything is successfully uploaded, open the Serial Monitor at a baud rate of 115200. Press the ESP32 EN/RST button, and it should print the ESP32 IP address.

![MPU6050-Web-Server-Serial-Monitor-IP-Address](https://github.com/user-attachments/assets/993ba734-1ccb-450c-83f2-35541048e05e)

## Demonstration
Open your browser and type the ESP32 IP address. You should get access to the web page that shows the sensor readings.

Move the sensor and see the readings changing as well as the 3D object on the browser.

![ESP-MPU6050-Web-Server-Arduino (1)](https://github.com/user-attachments/assets/537dd8a7-a7b4-4dbb-90d1-7e047b6b0d3e)

### Demo Video

https://github.com/user-attachments/assets/d525d0b9-a27f-45b3-bf2d-26cef4e26dde

**Note:** the sensor drifts a bit on the X axis, despite some adjustments in the code. Many of our readers commented that that’s normal for this kind of MCUs. To reduce the drifting, some readers suggested using a complementary filter or a kalman filter.
