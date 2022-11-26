# Bringup-of-AutonomousRoboticSoftwareStack-with-SNPE
## Introduction
The project demonstrates the integration of SNPE with Autonomous Driving/Robotics Software Stack on Qualcomm's Robotics Platform RB5 and  Object detection node created in Autoware.ai stack is integrated with Aslan,  an open-source full-stack software based on ROS framework for self-driving applications.

## Prerequisites 
- A Linux host system with Ubuntu 18.04. 
- Install Android Platform tools (ADB, Fastboot)  
- Download and install the SDK Manager for RB5 
- Flash the RB5 firmware image on to the RB5 
- Setup the Network on RB5. 
- Steps to Install Docker on RB5 
- Installed Docker on host system 
- Installed python3.6 on RB5 
- Installed python3.6 on host system 
- Installed ros-melodic-desktop-full 
- Catkin command line tools : catkin_tools 
- Colcon build 


### Installing Dependencies
- Docker Installation on RB5 
1. Prerequisites 
```sh
sh4.4 # apt-get update 
sh4.4 # apt-get install ca-certificates curl gnupg lsb-release 
```
2. Adding GPG Key 
```sh
sh4.4 # url -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archiv 
```
3. Adding repo URL in apt Sources 
```sh
sh4.4 # echo \ "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \ $(lsb_release -cs) stable" |  tee /etc/apt/sources.list.d/docker.list > /dev/null  
```
4. Installing Docker 
```sh
sh4.4 # apt-get update   
sh4.4 # apt-get install docker-ce docker-ce-cli containerd.io 
```

- Install ros-melodic-desktop-full on host system  
1. Setup your computer to accept software from packages.ros.org 
```sh
$ sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list' 
```
2. Setup keys 
```sh
$ sudo apt install curl 
$ curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add - 
```
3. First, make sure your Debian package index is up-to-date 
```sh
$ sudo apt update 
```
4. Desktop-Full Install: (Recommended): 
```sh
$ sudo apt install ros-melodic-desktop-full 
```
5. Environment setup 
```sh
$ source /opt/ros/melodic/setup.bash 
```

- Installing catkin-tools on host system 
1. First you must have the ROS repositories which contain the .deb for catkin_tools: 
```sh
$sudo sh \ 
 -c 'echo "deb http://packages.ros.org/ros/ubuntu `lsb_release -sc` main" \ 
 > /etc/apt/sources.list.d/ros-latest.list' 
$ wget http://packages.ros.org/ros.key -O - | sudo apt-key add - 
```
2. Once you have added that repository, run these commands to install catkin_tools: 
```sh
$ sudo apt-get update 
$ sudo apt-get install python3-catkin-tools 
```

- Installing ros-cross-compile on host system 
1. If you are using a Linux host, you must also install QEmu (Docker for OSX performs emulation automatically): 
```sh
$ sudo apt-get install qemu-user-static 
```
2. Install the stable release of ros_cross_compile tool 
```sh
$ pip3 install ros_cross_compile 
```
3. If you would like the latest nightly build, you can get it from Test PyPI 
```sh
$ pip3 install --index-url https://test.pypi.org/simple/ ros_cross_compile  
```

- Installing colcon build on host system 
```sh
$ sudo apt install python3-colcon-common-extensions 
```


### Steps to Setup the Setting up Aslan on host system  
1. Clone the Project ASLAN repository with submodules 
```sh
$ git clone --recurse-submodules https://github.com/project-aslan/Aslan.git 
```
`Note: if you haven't initialized and updated the submodules with --recurse-submodules when cloning, do` 
```sh
$ cd Aslan/ 
$ git submodule update --init –recursive 
```
2. Install additional dependencies 
```sh
$ cd Aslan 
$ rosdep install -y --from-paths src --ignore-src --rosdistro $ROS_DISTRO 
```
3. Initialize your catkin workspace 
```sh
$ catkin init 
```
4. Build your workspace using colcon build 
```sh
$ colcon build 
```
5. To start the Project ASLAN GUI, simply execute the bash script 
```sh
$ ./run 
```


### Cross compile Aslan on host system  
- Set up the workspace 
```sh
$ cd Aslan/.. 
```
`Note: if previously created and don’t want to use again run below command`
```sh
$ rm -rf cross_compile_ws/src 
$ mkdir -p cross_compile_ws/src 
$ cp -r /Aslan cross_compile_ws/src/ 
$ rsync -a /Aslan/../cross_compile_ws Aslan 
$ cd Aslan 
```

- Now run below command to invoke ros-cross-compilation tool, and this will give us Aslan cross compiler image 
```sh
$ ros_cross_compile cross_compile_ws --arch aarch64 --os ubuntu --rosdistro melodic 
```

- After build, run the following command in host system  
```sh
$ ls cross_compile_ws 
```

- After running the command, below directories will display if workspace is built successfully.  
```sh
cross_compile_ws 
|__build_aarch64/ 
|__cc_internals/ 
|__install_aarch64/ 
|__log/ 
|__src/ 
```

- To verify that the build created binaries for the target architecture (note "ARM aarch64" in below output. Your sha1 may differ) run below command: 
```sh
$ file cross_compile_ws/install_aarch64/waypoint_maker/lib/waypoint_maker/waypoint_saver 
```

- The binaries required for aarch64 architecture are created in the “install_aarch64” directory. 

- Create folder named core_perception in install_aarch64 directory. 
```sh
$ cd /PATH_TO/cross_compile_ws/install_aarch64/ 
$ sudo mkdir core_perception 
```

- Download source and copy vision_object_detect_snpe folder to  core perception folder using below command.  
```sh
$ sudo cp -r Object-Detection-with-SNPE-on-Autoware.ai/vision_object_detect_snpe cross_compile_ws/install_aarch64/core_perception/ 
```
`Note: vision_object_detect_snpe folder you can find in  /Object-Detection-with-SNPE-on-Autoware.ai after downloading source. `


### Push Docker image on Docker repository 
- Before pushing docker image, First verify docker image is created 	successfully with below command 
```sh
$ docker images 
```

- Should display cross compiled docker image, along  with dependent images.  As below: 
```sh
|__aarch64-ubuntu-melodic  
|__ros-cross-compile 
|__arm64v8/ubuntu 
|__ubuntu 
```

- Give a tag to the docker image 
```sh
$ docker tag USER/aarch64-ubuntu-melodic <USERNAME>/<REPOSITORY>: aarch64-ubuntu-melodic 
```

- To push docker image, first Login to docker hub by providing your id and password on the terminal. 
```sh
$ sudo docker login -u <USERNAME> -p <PASSWORD> 
```

- Push docker image on docker repository 
```sh
$ sudo docker push <USERNAME>/<REPOSITORY>: aarch64-ubuntu-melodic 
```


### Building the SNPE Python Wrapper for Object Detection 
- Go inside the src folder of project, 
```sh
root@ # cd <PROJECT_PATH>/src 
```
`Note: <PROJECT_PATH> is /install_aarch64/core_perception/vision_object_detect_snpe/`

- Copy the zdl folder from include directory of snpe sdk to our project source repositorie's include directory

- Run the command below in order to build the shared library for Python wrapper of the SNPE. 

```sh
root@ # g++ -std=c++11 -fPIC -shared -o qcsnpe.so qcsnpe.cpp -I /data/snpe/include/ -I Object-Detection-with-SNPE-on-Autoware.ai/vision_object_detect_snpe/include/zdl/ -I /usr/include/python3.6m/ -I /usr/local/lib/python3.6/dist-packages/pybind11/include -L /data/snpe/ -lSNPE `pkg-config --cflags --libs opencv` 
```


### Setup the install_aarch64 repository 
- Copy the /data/snpe/*.so to /opt/ros/melodic/lib/ 
- Copy the /install_aarch64/core_perception/vision_object_detect_snpe/src/qcsnpe.so to install/vision_object_detect_snpe/lib/vision_object_detect_snpe/ 

- Setup the environment variable LD_LIBRARY_PATH 
```sh
root@ # export LD_LIBRARY_PATH=/opt/ros/melodic/lib 
```

- Build the workspace 
```sh
root@ # cd /install_aarch64 
root@ # colcon build --packages-select vision_object_detect_snpe 
```

- Source the project environment using command  
```sh
root@ # source install/setup.bash 
```

- Save the changes made in Docker container by committing it with the command given below 
```sh
root@ # docker commit container_id image_name:tag 
```


### Running the Object Detection Node with ASLAN 
- Run the docker cross-compiled container inside RB5 
```sh
sh4.4 # docker run –it container_id bash 
```

- Go to the install_aarch64 directory 
```sh
root@ # cd /install_aarch64   
```

- Source the environment 
```sh
root@ # source install/setup.bash 
```

- Running the roslaunch file of Object Detection application, 
```sh
root@ # roslaunch vision_object_detect_snpe object_detect_snpe.launch  
```
