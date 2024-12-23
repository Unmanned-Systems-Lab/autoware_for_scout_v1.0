# 部署Autoware固定版本到Jetson AGX Orin

## 0.

**你的主机 ubuntu 至少要有40G的存储空间。**

**Useful resources**
- [Autoware Foundation homepage](https://www.autoware.org/)
- [Support guidelines](https://autowarefoundation.github.io/autoware-documentation/main/support/support-guidelines/)

## 1. 一般使用线路连接

- 供电：Type C（使用原装电源适配器）; DC 连接

  - Type C电源插口在网线端口这一端

- 显示：显示器 + DP接口 +显示器电源+ biaze DP转接口

- 外设：键盘 + 鼠标 两个USB

- 刷写：连接电脑的TypeC 端口与排插端口在同一侧

  <img src="https://github.com/Hikigaya-Yukina/Pictures/blob/main/interface.jpg" style="zoom:50%;" />

### 安插固态硬盘

<img src="https://github.com/Hikigaya-Yukina/Pictures/blob/main/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20240925102319.jpg" style="zoom:50%;" />

左边长条位置。取下固定螺丝，顺进逆出，插入对准，装上固定螺丝。

#### 给硬盘分区

刚刚插上全新的硬盘，读不到任何信息。

```
$ sudo fdisk -l 
```

识别到一个1.8TiB的disk

**“以固态硬盘为例分析/dev/nvme0n1p1**

 **/dev/nvme0n1p1表示什么? /dev表示设备。**

 **nvme0n1p1是一种硬盘格式，也叫Non-Volatile Memory Express（NVMe），是一种新型的高性能存储技术，它能够更快地从硬盘中读取和存储数据。**

 **NVMe技术使用PCIe接口，可以提供比传统SATA和SAS接口更高的传输速率，从而提高系统性能。NVMe硬盘的优势在于它可以提供更快的数据传输速率，而且可以支持更多的I/O操作，这使得它特别适合大型数据库系统，以及需要高性能的工作负载。**

 **/nvme0n1p1是指nvme硬盘的第一个分区，第一个字母n表示nvme，第二个字母1表示第一块硬盘，第三个字母p表示分区，最后的1表示第一个分区。“**

[参考文档1](https://blog.csdn.net/weixin_45761762/article/details/136123984)

[参考文档2](https://www.cnblogs.com/bpdhpm/p/11384473.html)

## 2.Orin 刷机

[参考文档1](https://zhuanlan.zhihu.com/p/632052753)

[参考文档2](https://blog.csdn.net/m0_54792870/article/details/129426560)

### 2.1 Nvidia SDK Manager

1. 下载SDK-manager的应用程序到ubuntu [官网链接](https://developer.nvidia.com/sdk-manager)

2. 注册Nvidia的账号密码

3. 在主机上安装下载下来的文件

   ```bash
   sudo apt install XXX.deb
   sudo apt-get update
   sudo apt-get upgrade
   ```

### 2.2 将Orin 布置为Recovery模式

首先确定已经和主机连接好了

<img src="https://i-blog.csdnimg.cn/blog_migrate/753cf027ceaeabdc461ee9dacaa33170.png#pic_center" alt="img" style="zoom:50%;" />

<img src="https://i-blog.csdnimg.cn/blog_migrate/f7d8a732f7baea35b9214016b60558ef.png#pic_center" alt="在这里插入图片描述" style="zoom:50%;" />

#### 2.2.1 Orin未开机

- 当处于未开机状态时，需要先长按住②键(Force Recovery键)，然后给Orin接上电源线通电，此时白色指示灯亮起，但进入Recovery模式后是黑屏的，所以此时连接Orin的显示屏不会有什么反应。


- 如果有已经装好的系统，直接关机orin，然后拔了电源，按住2，插上，一会儿松开


#### 2.2.2 Orin已开机

- 当处于已开机状态（系统关机接通电源）时，需要先长按住②键，然后按下③键(Reset键)，先松开③键，再松开②键。

#### 2.2.3 检查Orin

```bash
lsusb 
```

终端中输入后会出现NidiaCorp反馈

![image-20240920150712118](https://github.com/Hikigaya-Yukina/Pictures/blob/main/image-20240920150712118.png)

### 2.3 登陆SDK刷机

#### 2.3.1 确认设备型号

<img src="https://github.com/Hikigaya-Yukina/Pictures/blob/main/2099123201.jpg" alt="2099123201" style="zoom:70%;" />

一般会自动检测。如果不知道，请看下述

#### 2.3.2 查看设备型号

如果之前的orin上存在系统，则在orin上打开终端

```
lscpu
```

获取cpu信息（还是不知道）

或者

```
sudo apt-cache show nvidia-jetpack
```



或者查看内存大小

```
free -h
```

29 + 14 为 32G的吧



或者(但是怎么查看设备的SKU呢？没找到)

```
sudo apt install python3-pip
sudo -H pip3 install -U jetson-stats
sudo systemctl restart jtop.service
sudo jtop
```



或者

```
cat cat /etc/nv_boot_control.conf 
```

SKU 32G P3701-0000

64G P3701-0005

这是两个型号的区别

**如果看不到？**直接先预装一个ubuntu系统。

### 2.4 安装

#### 2.4.1 未检测到设备

![](https://i-blog.csdnimg.cn/blog_migrate/d6c6f034718c739c58c77d58e7a71eab.png#pic_center)

说明USB连接不正确，Orin没有处于恢复模式

#### 2.4.2 Continue

![image-20240920155336492](https://github.com/Hikigaya-Yukina/Pictures/blob/main/image-20240920155336492.png)

- 勾选Jetson 不勾选 Data Science
- 不勾选 Host Machine(这会造成安装主机版本的ubuntu，并让全部安装内容存储在主机上)
- 勾选 Target Hardware
- SDK Version 为 JetPack---6.0---rev.2
- 不勾选DeepStream
- 点击CONTINUE

#### 2.4.3 Accept and Continue

![image-20240920155445149](https://github.com/Hikigaya-Yukina/Pictures/blob/main/image-20240920155445149.png)

**全勾上。一定要勾CUDA等学习要使用的组件。**
**把OPENCV最新的组件(Computer vision)去掉勾选，因为是最新付费版本，与网上流通的有较大区别。**

当出现下列问题时，取消部分勾选：

- Docker烧录错误：docker 报错：container runtime with docker

  ```
  18:43:57 ERROR: NVIDIA Container Runtime with Docker integration (Beta) - target: E: Unable to locate package docker-ce-cli
  18:43:57 ERROR: NVIDIA Container Runtime with Docker integration (Beta) - target: E: Unable to locate package containerd.io
  18:43:57 ERROR: NVIDIA Container Runtime with Docker integration (Beta) - target: E: Couldn't find any package by glob 'containerd.io'
  18:43:57 ERROR: NVIDIA Container Runtime with Docker integration (Beta) - target: E: Couldn't find any package by regex 'containerd.io'
  18:43:58 ERROR: NVIDIA Container Runtime with Docker integration (Beta) - target: E: Unable to locate package docker-buildx-plugin
  18:43:59 ERROR: NVIDIA Container Runtime with Docker integration (Beta) - target: E: Unable to locate package docker-compose-plugin
  18:43:59 ERROR: NVIDIA Container Runtime with Docker integration (Beta) - target: E: Unable to locate package docker-ce-rootless-extras
  ```

  根据参考，将阿里云镜像加速地址放入

  如果多次烧录，且上述方案失败，则不勾选container runtime with docker

#### 2.4.4 设置密码

![image-20240920180920072](https://github.com/Hikigaya-Yukina/Pictures/blob/main/image-20240920180920072.png)

USER a123 PW 123(用户名必须小写开头)密码设置简单一些

然后点击Flash Orin注意和显示屏连接好，一会儿进行其他的设置。

#### 2.4.5 设置更新

![4061083089](https://github.com/Hikigaya-Yukina/Pictures/blob/main/4061083089.jpg)

32G  developer version；没有则32G version
ubuntu pc终端ping 192.168.55.1，确保ping通

## 3.程序安装

先联网

### 3.1 安装 浏览器 chorium

```text
sudo apt-get install chromium-browser -y
```

速度有些慢，但最好等待浏览器安装完成。否则会不可用。

### 3.2 安装 VPN

上github找clash-verge 下载arm64架构 linux系统的VPN

https://github.com/clash-verge-rev/clash-verge-rev/tree/main

梯子：

https://today.abyss.moe/

https://portal.shadowsocks.au/

### 3.3 安装ros2 和 vscode

本次要humble的版本用来配置autoware.universe的最新版本

使用鱼香ros进行该操作（更换系统源和第三方源）

```text
wget http://fishros.com/install -O fishros && . fishros
```

### 3.4 安装中文输入法

settings-> region&language-> manage installed languages

```
sudo apt install ibus
sudo apt install ibus-gtk ibus-pinyin
sudo apt install ibus-libpinyin
```

sudo reboot

再在keyboard input source中添加语言 pinying

更换语言 win+space

### 3.6 安装pip

```bash
sudo apt-get update
sudo apt-get install python-pip
sudo apt-get install python3-pip
sudo pip3 install jetson-stats
sudo jtop   # 启动jtop
```

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

