# Robotic "Headshot" Turret, Using YOLOv3 & ROS
### Full Parts-List, Installation, and Build Tutorial with Code.

This is an end-to-end build guide and installation tutorial with full parts/hardware listing, download for pretrained YOLOv3 weights for detecting "Human head", with Arduino and ROS source code to build a robotic pan/tilt turret that uses machine learning and relatively inexpensive hardware to autonomously detect and target human heads.  The code and weights files for this are hosted from a google drive download link included in this readme (simplifies download and install process), and you can see a few examples of it in action in the gifs below:


<img src="https://github.com/WyattAutomation/YOLOv3-ROS-Robotic-Head-Shot-Turret-/blob/master/orbbec.gif" height="400" width="285" align="left"> <img src="https://github.com/WyattAutomation/YOLOv3-ROS-Robotic-Head-Shot-Turret-/blob/master/lasershot.gif" height="400" width="510" align="right"> 
<p align="center">
  <img src="https://github.com/WyattAutomation/YOLOv3-ROS-Robotic-Head-Shot-Turret-/blob/master/kinectshot.gif" height="440" width="660">
</p>

## Credit to Other Projects and People
Of course, Joeseph Redmon's YOLOv3 is used in this project, which can be found here:
https://pjreddie.com/darknet/yolo/

The ROS Package for porting YOLOv3 to ROS used in this project is a fork of Legged Robotic's "darknet_ros" which can be found here:
https://github.com/leggedrobotics/darknet_ros

The ROS package for ros_astra_camera used in this project can be found here (ROS driver for the Orbbec Astra Pro):
https://github.com/orbbec/ros_astra_camera

And last, but certainly not least, the remaining 3 ROS packages I created for use in this project, were based on and/or derived from code included in Patrick Goebel's (aka Pirobot) "Robotics by Example, Volume 2".  If you like my project here, please go take a look at RBX2, and if you can find a copy of Patrick's book I highly, highly reccomend buying it as it will point you in the right direction for *many* different applications of robotics well beyond the scope of my project here.  I started off having next to no knowledge of robotics before reading his work, and it has provided the foundational knowledge needed in order for me to be successful in much of my work beyond this project as well:
https://github.com/pirobot

Also, I will include a previous project of my own, where I document how to create the weights I use in this project for YOLOv3 from Google's Open Images Dataset:
https://github.com/WyattAutomation/Train-YOLOv3-with-OpenImagesV4

## Getting Started 

##### NOTE REGARDING OS:  This guide assumes you are using a brand new, untouched installation of Xubuntu 18.04.  This install process likely works just fine on Ubuntu 18.04 with the defualt Gnome desktop too, but I cannot promise it will work on anything else beyond Ubuntu/Xubuntu 18.04.  This is not built for Windows AT ALL, nor do I ever plan to support a Windows installation for this in the future (the next iteration in developement is for ROS2/Xubuntu 20.04).  With that being said, this document assumes you are starting with a fresh installation of Xubuntu 18.04 (my only suggestion for Windows users is to just buy a cheap SSD to dedicate to this if you really want to try this build). 

### Download and untar ROS_YOLO_headshot_main.tar.gz, containing all source code and files needed for this project, as well as the pretrained weights for detecting "Human head" from here:

##### (If you are just looking for the pretrained YOLOv3 weights for "Human head", they are located at "headshot_main/catkin_ws_src_files/darknet_ros/darknet_ros/yolo_network_config/weights/backup/" from the below Google Drive download link) 

https://drive.google.com/drive/folders/1P23rZR8ul1poW_VWSHgcgzApEtZsTR-6?usp=sharing

-After downloading from the above link, open a terminal and run the following to extract:

```
cd ~/Downloads/
tar -xvf ROS_YOLO_headshot_main.tar.gz 
```

-This should extract everything to a folder at ~/Downloads/headshot_main


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

I've tested this setup process from start to finish as of April 26 2020 with Xubuntu 18.04 and can confirm it works without issue.  You can download it from here, and simply burn the iso to a thumb drive using rufus or Etcher: https://xubuntu.org/release/18-04/

*When installing Xubuntu 18.04, uncheck installing any additional software in order to avoid issues with the wrong Nvidia drivers being installed.  Also, be careful to download 18.04 and not the newest 20.04!  I am working on a ROS2 improvement of this project that may be released in the near future, that will work with 20.04, but for now, stick with 18.04.*

Get a "vanilla" installation of Xubuntu 18.04 running on a dedicated SSD, then proceeed to the next steps.

#### Arbotix controller and Arduino:
Assemble the turret kit hardware, and plug in the FTDI cable to the ArbotiX controller and the other end to a USB (2 or 3, doesn't matter) port on your Xubuntu PC, then plug in the power supply to the barrel connector on the Arbotix.

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

-Go to File->Upload to upload the "ros" sketch to your ArbotiX board.  The code from this sketch may be commented to mention that it is for the ax-12a Dynamixels, but it works just as well for the 18s.  

NOTES on ArobotiX/Dynamixels/Arduino:
"/dev/ttyUSB0" for the serial port is the default device that the USB FTDI cable coming out of your ArbotiX board should show up as.  If this is different, please open an issues thread and I'll provide docs on what needs to be done to change it, as that is set up in the ROS packages and has to be changed there too if it is different.

The drivers, libraries, and code for the ArbotiX Arduino Sketch above are from Trossen's quickstart guide here, though I would highly suggest NOT following it for the purpose of my project:
https://learn.trossenrobotics.com/index.php/getting-started-with-the-arbotix/7-arbotix-quick-start-guide

You can unplug the power supply to the ArbotiX with a reasonble amount of expection of it not ruining the board, if it's frozen, if the robot goes wildly out of control etc.  I have put all of this equipment through a significant amount of abuse and it is not fragile to physical strain or sudden loss of power (though I would avoid exposure to moisture/water/liquid).

####  Xubuntu 18.04 Nvidia Driver and CUDA installation
 
-Open a terminal and run the following commands to install the NVidia Drivers and CUDA 10.1

```
sudo apt update
sudo add-apt-repository ppa:graphics-drivers/ppa

sudo apt-key adv --fetch-keys  http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub

sudo bash -c 'echo "deb http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/cuda.list'

sudo bash -c 'echo "deb http://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/cuda_learn.list'

sudo apt update
sudo apt install cuda-10-1
sudo apt install libcudnn7

```

-Add CUDA library paths to ~./bashrc and reboot
```
echo "export PATH=/usr/local/cuda-10.1/bin:$PATH" >> ~/.bashrc
echo "export LD_LIBRARY_PATH=/usr/local/cuda-10.1/lib64:$LD_LIBRARY_PATH" >> ~/.bashrc
sudo reboot
```
-Check to make sure Nvidia driver installed after rebooting:
```
nvidia-smi
```
-Check CUDA installation too:
```
nvcc --version
```

#### Install ROS Melodic:

Run the following in terminal:

-Set up package sources for ROS:
```
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
sudo apt update

```

-Install ROS melodic (takes a while):
```
sudo apt install ros-melodic-desktop-full
```

-set up .bashrc to auto source ROS installation, and source it in current terminal:
```
echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

-Install ROS package tools and rosdep for python:
```
sudo apt install python-rosdep python-rosinstall python-rosinstall-generator python-wstool build-essential
sudo apt install python-rosdep
```

-Initialize ROS and update rosdep:
```
sudo rosdep init
rosdep update
```

#### Create New Catkin Workspace, Copy, and Build Sources for Headshot Project

-Make a new catkin workspace where the code for this project will be placed:
```
mkdir -p ~/catkin_headshot/src
cd ~/catkin_headshot/
catkin_make
```

-Add the workspace to .bashrc and source it:
```
echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
source ~/.bashrc
```


-Install remaining dependencies for ROS packages:
```
sudo apt install ros-melodic-libuvc-camera ros-melodic-arbotix ros-melodic-rgbd-launch
```

-Copy ROS Packages from the downloaded/extracted folder to the new catkin workspace and build it:
```
cp -r ~/Downloads/headshot_main/catkin_ws_src_files/. catkin_headshot/src/
cd ~/catkin_headshot/
catkin_make -DCMAKE_BUILD_TYPE=Release
```

-Copy the calibration files for the Astra Pro to home/.ros:
```
cp -r ~/Downloads/headshot_main/camera_info/ ~/.ros
```


-Add udev rules for the Orbbec Astra Pro, and rebuild Astra Pro ROS package:
```
roscd astra_camera
sudo cp 56-orbbec-usb.rules /etc/udev/rules.d
sudo service udev reload
sudo service udev restart
catkin_make --pkg astra_camera -DCMAKE_BUILD_TYPE=Release
```


-Add your user to "dialout" group to have permissions for the Arbotix via USB:
```
sudo usermod -a -G dialout $USER
```


-Make head_tracker.py and headtrack.py nodes executable:
```
roscd turret_controller/
chmod +x nodes/head_tracker.py 
roscd yolo_targeting/
chmod +x nodes/head_track.py
```


*It's often a good idea to unplug the astra from whatever usb port it's in and move it to another here, and reboot.  Sometimes the ROS package for the Astra Pro can't find the device until after plugging it back into a different port and/or reboot.*

-After reboot, run the headshot.py script in headshot_main:
```
python ~/Downloads/headshot_main/headshot.py
```

You may see warnings and several other things scroll quickly accross the terminal screen before landing at the collection of windows that look like they do in the screenshot below; with the pan/tilt turret now tracking the nearest detected "Human head"!


## NOTES/Manual Launch Instructions/Debugging:

The Obbecc Astra Pro is *quite* finicky about wanting to sometimes be the last device that was plugged in.  If it's non-responsive, even after you've confirmed the udev rules are set correctly, try plugging it in to a different USB 3.0 port on a different bus, and reboot.  Replugging after a reboot sometimes helps too. 

If you have any issues with this system not working from the headshot.py script, you can run each roslaunch command that the turret uses in seperate terminals for debugging, or for development purposes.  

#### If you are debugging or wish to run the roslaunch commands one-by-one instead of through the script:

*-This command has to be run first, or none of the others will work properly!!*  This is for initializing roscore, the connection to the Arbotix board, and certain parameters of the actuators.  After the successful catkin_make earlier in the guide, the only associated file to adjust for this command (if there are issues with it) is the launchfile at "~/catkin_headshot/src/turret_init/launch/"
```
roslaunch turret_init turret_init.launch
```

-With the previous command still running, open another terminal and run this one directly after (the darknet and yolo_targeting packages both require this to be running first).  This is the launchfile to initialize the Astra Pro Camera:
```
roslaunch astra_camera astrapro.launch
```

-With the previous two roslaunch commands still running, open another terminal and run the next roslaunch command, which should center the servos and awaits the published "target" topic from the "yolo_targeting" package/node to tell it where to point the servos (via pose-stamped messages):
```
roslaunch turret_controller head_tracker.launch
```

-Open yet another terminal with the previous commands still running, and start the ros_darknet node, which I've preconfigured to load the weights for "Human head".  This node subscribes to the rgb messages from the "/camera/rgb/image_raw" topic published by the Astra Camera node.  You can find this configured in "~/catkin_headshot/src/darknet_ros/darknet_ros/launch/ros_headshot.launch"   I have already put the weights in the proper directory for this, as well as preconfigured the .yaml files, the yolo network config, launchfile and other related components to work for detecting "human head".  Should you have issues with this node, the ros_darknet issues thread in the "credits" section at the start of this repo is a good place to start.  Roslaunch command for this package/node is:
```
roslaunch darknet_ros darknet_ros_headshot.launch
```

-Finally, with all of those above commands running, run the roslaunch command in another new terminal from the "yolo_targeting" package.  This launches the "head_track.py" node from the yolo_targeting package in your src folder in the catkin worksapce, which subscribes to the bounding box data topic published by the ros_darknet node and the depth registered pointcloud of the astra pro, uses the bounding box data to get the center x,y position of the object detected in the depth registered rgb image, converts that to the u,v 2D position of the center of the detected object in the depth frame, pulls the related 3D xyz data associated with the depth frame, and converts that to a pose stamped messaged that it then publishes for the turrent_controller/headtracker package/node to subscribe to.
```
roslaunch yolo_targeting head_track.launch
```

(I screwed up a little here with the naming and named the launchfile for this "head_track" when there's already a launchfile from the turrent_controller package named "head_tracker", but keep in mind they do completely different things).


If there are any questions on using this with a Kinect, I will be updating it soon to include instructions for this.  You basically only need to change parameters for the ros_darknet node to subscribe to the image topic (which has a different name), and instead of the astra_pro_camera package, you'll use the freenect_stack package from another repo.  I will update this soon to include these instructions.

Have fun, don't do anything stupid, and YOLO, robot style! (v3 at least)..
