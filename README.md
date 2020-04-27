# YOLOv3/ROS Robotic Pan/Tilt "Headshot" Turret
### Full Parts-List, Installation, and Build Tutorial with Code.

This is an end-to-end build/installation tutorial with a full parts/hardware list, pretrained YOLOv3 weights for "Human head", and ROS packages to help create your own robotic pan/tilt turret that uses machine learning and relatively inexpensive hardware to autonomously "target" and track human heads.  You can see a few examples of it in action in the gifs below:    
<img src="https://github.com/WyattAutomation/YOLOv3-ROS-Robotic-Head-Shot-Turret-/blob/master/orbbec.gif" height="400" width="285" align="left"> <img src="https://github.com/WyattAutomation/YOLOv3-ROS-Robotic-Head-Shot-Turret-/blob/master/lasershot.gif" height="400" width="510" align="right"> 
<p align="center">
  <img src="https://github.com/WyattAutomation/YOLOv3-ROS-Robotic-Head-Shot-Turret-/blob/master/kinectshot.gif" height="440" width="660">
</p>

## Credit to other projects
Of course, Joeseph Redmon's YOLOv3 is used in this project, which can be found here:
https://pjreddie.com/darknet/yolo/

The ROS Package for porting YOLOv3 to ROS used in this project is a fork of Legged Robotic's "darknet_ros" which can be found here:
https://github.com/leggedrobotics/darknet_ros

The ROS package for ros_astra_camera used in this project can be found here (ROS driver for the Orbbec Astra Pro):
https://github.com/orbbec/ros_astra_camera

And last, but certainly not least, the remaining 3 ROS packages I created for use in this project, were based on and/or derived from code included in Patrick Goebel's (aka Pirobot) "Robotics by Example, Volume 2".  If you like my project here, please go take a look at RBX2, and if you can find a copy of Patrick's book I highly, highly reccomend buying it as it will point you in the right direction for *many* different applications of robotics well beyond the scope of my project here.  I started off having next to no knowledge of robotics before attempting this project, and his literature provided nearly everything I needed in order to be successful here.  I was able to become reasonably fluent with ROS in about two weeks via his documentation:
https://github.com/pirobot

Also, I will include a previous project of my own, where I document how to create the weights I use in this project for YOLOv3 from Google's Open Images Dataset:
https://github.com/WyattAutomation/Train-YOLOv3-with-OpenImagesV4

## Getting Started

### Download and untar ROS_YOLO_headshot_main.tar.gz, containing all source code and files needed for this project, as well as the pretrained weights for detecting "Human head" from here:


### Hardware Needed

*NOTE: Though I am currently working on an ARM version of this system for the Nvidia Jetson Nano (I have it 'functional', but not optimized well enough to reproduce the results in the gifs above), this specific build tutorial is for x86/PC only!*

Let's get started with the hardware needed to make this work

#### Robotics Components:

I used a "Phantom X" turret kit from Trossen Robotics, with 2 AX-18A actuators, and an ArbotiX controller.  While the code from this repo should work just fine with AX-12A actuators, I would highly reccomend spending the extra $100 and selecting the 18s when you order, as I cannot attest to the performance of the AX-12As.  The 18s seem to have no problem slinging around 6lbs+ at high speed, and at about $300 USD for a decent kit with an arduino compatible controller, it's worth the money.  Make sure whatever variant that you select, that it comes with the Arbotix Controller included with this kit:
https://www.trossenrobotics.com/p/phantomX-robot-turret.aspx

#### PC:

The PC I used for running ROS/YOLOv3 is a quad-core Ryzen 3, with an Nvidia GTX 1060 (6GB VRAM), with 8GB of system memory.  As long as you have an NVidia GPU from roughly 2018 or newer with 4GB+ of VRAM you should be fine; however you may run into CUDA compatibity issues with legacy GPUs and I'm not going to help resolve issues of that category.  If you can't get CUDA 10.1 installed on your system via the instructions provided here, I'm not going to help address it as it is outside the primary scope of this document.  With that being said, this doesn't require anything spectacular to get YOLO running in ROS at about 30 FPS, just try to keep in mind that older legacy Nvidia GPUs may not work.

#### Depth Camera:

For this tutorial, I will be including installation instructions for using an Orbbec Astra Pro infrared depth camera.  I will update it later to include the Xbox 360 Kinect (which is actually significantly easier to install, but slightly less performant).  You can find information about Orbbec depth cameras from their website, and typically can find an Astra Pro pretty easily on Ebay or Amazon.  I bought mine on Ebay for about $90:
http://shop.orbbec3d.com/Astra-Pro_p_35.html

#### Hardware Summary:

1x Trossen Robotics "PhantomX" Robot Turret kit:
```
    1x ArbotiX Robocontroller
    2x AX-12A or AX-18A Dynamixel Actuators
    Black Bioloid Frames F2, F3
    Dynamixel Robot Turret Base & Hardware Kit
    1x 12v 2amp Power Supply
    1x FTDI USB Cable
```
1x Orbbec Astra Pro Camera

Desktop/Laptop PC:
```
    NVidia GPU, 4GB+ VRAM minimum, reccomended 2018 or newer
    At least a quad-core, 3.0GHz per-core CPU, AMD or Intel
```

## Installation

#### OS:

This guide is for Ubuntu Linux *only* (specifically, Xubuntu 18.04)!! *This project will not work on Windows, nor will I provide support on issues with Windows systems!!!!*

I would highly reccomend starting with a clean installation of Xubuntu 18.04, and cannot guarantee the functionality of this on anything else.  I've tested this setup process from start to finish as of April 26 2020 with Xubuntu 18.04 and can confirm it works without issue.  I'd reccomend buying a cheap 120GB SSD to dedicate to this project.

*When installing Xubuntu 18.04, uncheck installing any additional software in order to avoid issues with the wrong Nvidia drivers being installed.  Also, be careful to download 18.04 and not the newest 20.04!  I am working on a ROS2 improvement of this project that may be released in the near future, that will work with 20.04, but for now, stick with 18.04.*

Get a "vanilla" installation of Xubuntu 18.04 running on a dedicated SSD, then proceeed to the next steps.

#### Setting up Arbotix controller and Arduino:
Once you've built the physical turret kit, plug in the FTDI cable to the Arbotix and the other end to USB port on your PC, then plug in the power supply to the barrel connector on the Arbotix.

Install dependencies for Arduino:

-Install Java 8 jdk for the older version of Arduino we are going to use:
```
sudo apt install openjdk-8-jdk
```
-Install libusb-0.1-4:
```
sudo apt-get install libusb-0.1-4
```

-Open a terminal, cd to the arduino-1.0.6 folder from the downloaded and extracted "headshot_main" folder, and run arduino:
```
cd ~/Downloads/headshot_main/arduino-1.0.6
sudo bash arduino 
```
-Select "no" on "new version" install prompt (if asked), when the Arduino program opens

In Arduino:

-Go to File->Sketchbook->ros and open it

-Go to Tools->Board and make sure "ArbotiX" is selected

-Go to Tools->Serial Port and make sure /dev/ttyUSB0 is selected
(NOTE: /dev/ttyUSB0 is the default device that the USB FTDI cable coming out of your ArbotiX board should show up as.  If this is different, please open an issues thread and I'll provide docs on what needs to be done to change it) 

-Go to File->Upload to upload the "ros" sketch to your ArbotiX board.  The code from this sketch may be commented to mention that it is for the ax-12a Dynamixels, but it works just as well for the 18s.  

The drivers, libraries, and code for the ArbotiX Arduino Sketch above are from Trossen's quickstart guide here, though I would highly suggest NOT following it for the purpose of my project:
https://learn.trossenrobotics.com/index.php/getting-started-with-the-arbotix/7-arbotix-quick-start-guide

You can unplug the power supply to the ArbotiX with a reasonble amount of expection of it not ruining the board, if it's frozen, if the robot goes wildly out of control etc.  I have put all of this equipment through a significant amount of abuse and it is not fragile to physical strain or sudden loss of power (though I would avoid exposure to moisture/water/liquid).


 
