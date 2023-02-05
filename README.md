---
name: "@Abubakarharuna10"
project: "Artificial Rescuer"
---

# Artificial Rescue

## Summary

With the increasing threat of climate change, greater populations and consequently, more extreme environmental events
the need for better emergency rescue services is paramount
As time moves forward, natural disasters will continue to wreak havoc on cities and counties around the world,
causing many people to die. Bolstering the power of search and rescue teams with autonomous UAVs that can identify people
in disaster relief zones could significantly reduce the impact tornadoes, earthquakes, tsunamis, floods, etc...
My solution to strengthening search and rescue operations is to outfit an autonomous unmanned aerial vehicle (UAV)
with a computer vision system that detects the location of people as the vehicle flies over ground. The powerful neural-network capabilities
of the Jetson Nano Dev Kit will enable fast computer vision algorithms to achieve this task. Joseph Redmon's YOLOv3 algorithm will be used
to do the actual object-detection (people) in the camera's view While the UAV is flying a waypoint mission using ArduPilot, PX4, or any other autonomous flight control stack, the absolute location of people in the camera view can be calculated based on the altitude, orientation, and GPS location of the UAV.

## Step-by-Step guide for Running the App

#### Step 1

clone the repository

```
git clone https://github.com/Abubakarharuna10/artificial_rescuer.git
```

#### Step 2

```cd artificial_rescuer

```

#### Step 3

Run this file


```
./setup
```

### Darknet Installation

Now that the project code is ready, you will need to install the actual computer vision code. This project uses Joseph Redmon's Darknet tiny-YOLOv3 detector because of its blazingly-fast object detection speed, and small memory size compatible with the Jetson Nano Dev Kit's 128-core Maxwell GPU.

### Clone the Darknet GitHub repository inside of the artificial_rescuer GitHub repository cloned in the previous section

```
git clone https://github.com/pjreddie/darknet.git
cd darknet

```

#### Because Darknet runs "like 500 times faster on GPU, " (Joseph Redmon) we will modify the Makefile for compiling with GPU support. Using a text editor, modify the flags at the top of the Makefile to match the following, leaving any other values as they are

```
GPU=1
CUDNN=1
OPENCV=1
```

##### Now that the Makefile is corrected, compile Darknet using the following command

```
make

```

#####  If the compilation was successful, there should be a file called libdarknet.so in the Darknet repository. Copy this file to the jetson-uav directory so the script will have access to it.

```
cp libdarknet.so ..
```

##### Install npm
Luckily, you do not need to spend hours or days training the YOLOv3 detector because I pre-trained the model on a Compute Engine instance on Google's Cloud Platform. The model was trained on the 2017 COCO dataset for around 70 hours using an NVIDIA Tesla V100, and the weights (eagleeye.weights) are saved in the GitHub repository for this project.

If you would like the train the model further, or understand how training YOLOv3 works, I recommend reading this article, as it helped me greatly during the process.

keypoints-splash_sCUuVLUZ9B.avif

- Setup Script for Auto-launch at Startup

In order for the code to run as seamlessly as possible, the script needs to be setup to run at startup on the Jetson Nano. Rather than waiting to launch the code via an SSH session, this will allow for the Dev Kit to be powered on and automatically begin detecting people in frame while the UAV is flying on a mission.

The auto-launch capability will be achieved by setting up a systemd service (eagleeye.service) that runs a bash file (process.sh), which then runs the python script (main.py) and streams the output to a log file (log.txt). It may sound complicated, but only a few simple steps is all it takes to get it up and running!

##### Step 1
Modify the second line of process.sh to match where you cloned the jetson-uav GitHub repository. Make sure to only change the path that is shown in bold below, as the other files are relative to this path.
```
#!/bin/bash
cd /home/learner/Desktop/artificial_rescuer && python3 -u main.py > log.txt
```
##### Step 2
Next, modify line 6 of eagleeye.service to match the location of the process.sh file used in the previous step. (Shown in bold below)
```
  [Unit]
Description=EagleEye service

[Service]
User=root
ExecStart=/home/learner/Desktop/artificial_rescuer/process.sh

[Install]
WantedBy=multi-user.target
 ```

##### Step 3
Copy eagleeye.service to the /etc/systemd/system directory so systemd has access to it.


```
 sudo cp eagleeye.service /etc/systemd/system/
```
##### Step 4
Enable the newly-created systemd service, so it will automatically run at startup.


```
sudo systemctl enable eagleeye
```
#### Jetson Nano as Wi-Fi Access Point Configuration

While this section is optional, I highly recommend using a USB Wi-Fi module to setup the Jetson Nano as a Wi-Fi access point so you can connect to it while not tethered to an ethernet connection. This will allow you to monitor the processes running on the Jetson Nano for debugging and ensuring no errors occur while your UAV is either preparing for flight, in flight, or landed.

#### Step 1
Ensure a USB Wi-Fi module is plugged into one of the Dev Kit's USB ports. I recommend the Edimax EW-7811Un 150Mbps 11n Wi-Fi USB Adapter attached in the Hardware components section of this project.

##### Step 2

 In a terminal on the Jetson Nano, run the following command to create an access point with an SSID and password of your choice. The second command will enable auto connect functionality so the Jetson Nano will automatically connect to its hosted network if there is no other network option available.

```
nmcli dev wifi Hotspot ifname wlan0 ssid <SSID> password <PASSWORD>
nmcli con modify Hotspot connection.autoconnect true
```

#### Camera Calibration

The camera calibration process will allow for the removal of any distortion from the camera lens, providing more accurate location estimates of people in frame while in flight. Camera calibration will be done by taking multiple pictures of a calibration chessboard at various positions and angles, then letting the script find the chessboard in each image and solve for a camera matrix and distortion coefficients.

Print out a 10 by 7 chessboard and adhere it to a small rigid surface. The calibration script will search for this marker in each image. It is vital that the chessboard is 10 by 7, as the script will look for the interior corners of the chessboard which should be 9 by 6

##### Step 3

Run the calibrate.py file in the downloaded repository using the following command. This process will allow you to save multiple pictures of a chessboard to a desired location (a folder named "capture" in this case).

```
python3 calibrate.py capture

```
Press [spacebar] to save a picture of the chessboard in various positions and angles as shown below. The more variability in chessboard images, the better the calibration will be.

#### Step 4

Run the calibration script again, instead adding the -c flag to run calculation rather than saving new images. (Use the same capture path from running the previous time.)

```
python3 calibrate.py -c capture
```
This process will look through all captured images, detecting the chessboard corners in each one; any image that it could not find the chessboard in will be deleted automatically. After finding all corners in each image, the script will execute OpenCV's calibrateCamera routine to determine the camera matrix and distortion coefficients. The calibration parameters will be saved to a file (calibration.pkl) for use by the main script.

##### I will show you how to Assemble the Jetson Module when my device's arrive from amazon
