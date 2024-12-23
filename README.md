# 部署Autoware固定版本到Jetson AGX Orin

## 4. 安装autoware.universe(humble)

[官方文档](https://autowarefoundation.github.io/autoware-documentation/main/installation/autoware/source-installation/)

按照官方的安装顺序就行了。**但先看4.1**，再安装

### 4.1 How 2 set up a development environment 中需要替换和注意的部分

#### 4.1.1 抓取的库的更新

1. Clone `autowarefoundation/autoware` and move to the directory.

```
git clone https://github.com/Unmanned-Systems-Lab/autoware_for_scout_v1.0.git
cd autoware
```

这里使用了稍早的版本进行部署，确保该版本不会被官方更新，而且已经移殖到github仓库里了。



#### 4.1.2 暂时跳过CUDA、cuDNN、TensorRT的安装。

Jetson Orin在Nvidia的官网上找不到对应版本的、arm64的、jetsib AGX的、这些组件的安装包。要使用对应功能需要在内部进行更改。

#### 4.1.3 dev_tools

clang-format==${pre_...}$换成clang-format==17.0.5

#### 4.1.4 geographiclib 安装

过慢的时候去官网下载对应的安装包

#### 4.1.5 ROS2 dev tools安装

[20.04 Galactic版本](https://blog.csdn.net/zardforever123/article/details/132029636?spm=1001.2014.3001.5502) 参考该文档

同时安装这个setuptools:

```bash
pip install setuptools==58.2.0
```

需要使用该版本。

#### 4.1.6 忽略的地方

![](https://github.com/Hikigaya-Yukina/Pictures/blob/main/Screenshot%20from%202024-12-20%2017-43-43.png)

Note下这几个先不管。

### 4.2 How 2 set up a workspace 需要注意的地方

按照官网步骤做就行了。这一段结束后就结束了。

#### 4.2.1 vcs import src < autoware.repos

安装时要确认全部都拉取了，要注意有没有红色的报错内容。否则再拉取一次。

#### 4.2.2 安装ROS packages

遇到报错参考[20.04 Galactic版本](https://blog.csdn.net/zardforever123/article/details/132029636?spm=1001.2014.3001.5502) 

### 4.3 colcon build 中 可能存在的报错

#### 4.3.1 angles/angles/angles.h无法找到

修改nebula_decoders这个文件夹里的CMakeLists

include_directories 里直接加入文件绝对路径（/opt/ros/humble/include)

#### 4.3.2 diognostic.h 无法找到

修改multi_object_tracker里的CMakeLists.

include_directories 里直接加入文件绝对路径（/opt/ros/humble/include)

#### 4.3.3 FindPACP.cmake 不存在
根据[该方法](https://blog.csdn.net/u013834525/article/details/96843094)安装功能包，并写一份PACP.cmake放入对应文件夹

## 5.使用autoware的仿真

现在下载下来的东西基本都调好了。

根据[Tutorials](https://autowarefoundation.github.io/autoware-documentation/main/tutorials/ad-hoc-simulation/planning-simulation/)的教程，下载官方地图进行运动仿真。各种操作在官方的Tutorials中已经写好了

### 5.1 官方运动仿真需要更改的命令

- ros2launch中对应的车辆和传感器模型改为：

  ```bash
  vehicle_model:=scout_vehicle sensor_model:=scout_sensor_kit
  ```


- 修改对应位置的controllor_mode: 由mpc改为pure_pursuit

  ![](https://github.com/Hikigaya-Yukina/Pictures/blob/main/Screenshot%20from%202024-12-23%2010-15-11.png)

- 在planning_simulator.launch.xml文件中，要把地图名字更改成官方资源包的地图名称：

```
src/launcher/autoware_launch_for_scout/autoware_launch/launch/planning_simulator.launch.xml

把<!--Map-->下面这一行的default 的值改为官方地图的名称。
```

然后

```bash
source install/setup.bash
```

ros2launch 对应文件，通过官方教程进行学习。

### 5.2 扫描的已经写好的地图的启动

```bash
ros2 launch autoware_launch autoware.launch.xml map_path:=$HOME/autoware_map_test vehicle_model:=scout_vehicle sensor_model:=scout_sensor_kit lateral_controller_mode:=pure_pursuit
```

需要下载对应[地图](https://pan.baidu.com/s/1tLyoF9_isfyJxaLTkd2w5w?pwd=4ct3)，百度云网盘。下载后解压在HOME里。

同上，需要更改对应位置的地图名称。

## 6. 实车启动

小车线路连接。连接好后作出如下更改。

### 6.1 CAN通讯挂载(只需要设置一次）

当前版本由于Nvidia烧录的内核没有包含usb_CAN的东西，故采用外接，使用RX TX的端口 进行CAN通讯。
编写使用的脚本：(根据需求修改这些参数)
```bash
sudo vim /home/[你的用户名]/CAN_scripts/CAN.sh
#你也可以使用gedit
```
脚本的内容：
```bash
#!/bin/bash
sudo busybox devmem 0x0c303000 32 0x0000C400
sudo busybox devmem 0x0c303008 32 0x0000C458
sudo busybox devmem 0x0c303010 32 0x0000C400
sudo busybox devmem 0x0c303018 32 0x0000C458
sudo modprobe can
sudo modprobe can_raw
sudo modprobe mttcan
sudo ip link set down can0
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0
```
编辑并保存后设置权限：
```bash
sudo chmod +x /home/[你的用户名]/CAN_scripts/CAN.sh
```
使用systemd服务
```bash
sudo vim /etc/systemd/system/CAN.service
#也可以使用gedit
```

在文档中编辑
```txt
[Unit]
Description=CAN service
 
[Service]
ExecStart=/home/[你的用户名]/CAN_scripts/CAN.sh
 
[Install]
WantedBy=multi-user.target
```

保存并关闭该文件，然后启动该服务并将其设置为开机自启：
```bash
sudo systemctl daemon-reload
 
sudo systemctl start CAN.service
 
sudo systemctl enable CAN.service
```

如果要检查状态：
```bash
sudo systemctl status CAN.service
```

如果要停止服务：
```bash
sudo systemctl stop CAN.service
 
sudo systemctl disable CAN.service
```
使用candump can0 检查通讯连接是否建立
### 6.2 通讯节点启动
```bash
cd autoware_for_scout_v1.0
source install/setup.bash
ros2 launch scout22autoware_interface interface_scout.launch.py
```

### 6.3 autoware 启动
```bash
#在launch文件里设置了pure_pursuit后不需要再在末尾加上 lateral_controller_mode:=pure_pursuit
 ros2 launch autoware_launch autoware.launch.xml map_path:=$HOME/autoware_map_test vehicle_model:=scout_vehicle sensor_model:=scout_sensor_kit
```
