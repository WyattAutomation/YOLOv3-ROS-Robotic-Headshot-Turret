# YOLOv3/ROS Robotic Pan/Tilt "Headshot" Turret
### Full Parts-List, Installation, and Build Tutorial with Code.

This is an end-to-end build/installation tutorial with a full parts/hardware list, pretrained YOLOv3 weights for "Human head", and ROS packages to help create your own robotic pan/tilt turret that uses machine learning and relatively inexpensive hardware to autonomously "target" and track human heads.  You can see a few examples of it in action in the gifs below:    
<img src="https://github.com/WyattAutomation/YOLOv3-ROS-Robotic-Head-Shot-Turret-/blob/master/orbbec.gif" height="400" width="285" align="left"> <img src="https://github.com/WyattAutomation/YOLOv3-ROS-Robotic-Head-Shot-Turret-/blob/master/lasershot.gif" height="400" width="510" align="right"> 
<p align="center">
  <img src="https://github.com/WyattAutomation/YOLOv3-ROS-Robotic-Head-Shot-Turret-/blob/master/kinectshot.gif" height="440" width="660">
</p>

## Getting Started

### Hardware

*NOTE: Though I am currently working on an ARM version of this system for the Nvidia Jetson Nano (I have it 'functional', but not optimized well enough to reproduce the results in the gifs), this specific build tutorial is for x86 PC only!*

Let's get started with the hardware needed to make this work

For the Pan/Tilt Turret Robot itself:

I used a "Phantom X" turret kit from Trossen Robotics, with 2 AX-18A actuators, and an ArbotiX controller.  While the code from this repo should work just fine with the AX-12A actuators, I would highly reccomend spending the extra $100 for the 18s as I cannot attest to the performance of the AX-12As.  The 18s seem to have no problem slinging around 6lbs+ at high speed, and at about $300 USD for a decent kit with an arduino compatible controller, it's worth the money.  
https://www.trossenrobotics.com/p/phantomX-robot-turret.aspx

For the PC:

The PC I used for running ROS/YOLOv3 is a quad-core Ryzen 3, with an Nvidia GTX 1060 (6GB VRAM), with 8GB of system memory.  As long as you have an NVidia GPU from roughly 2018 or newer with 4GB+ of VRAM you should be fine; however you may run into CUDA compatibity issues with legacy GPUs and I'm not going to help resolve issues of that category.  If you can't get CUDA 10.1 installed on your system via the instructions provided here, I'm not going to help address it as it is outside the primary scope of this document.  With that being said, this doesn't require anything spectacular to get YOLO running in ROS at about 30 FPS, just try to keep in mind that older legacy Nvidia GPUs may not work.

For the Depth Sensor:

