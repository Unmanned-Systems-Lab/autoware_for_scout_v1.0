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

### 4.3 conlon build 中 可能存在的报错

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

