# livox mapping

![image-20241209161847378](/home/wenming/.config/Typora/typora-user-images/image-20241209161847378.png)

![image-20241209161902542](/home/wenming/.config/Typora/typora-user-images/image-20241209161902542.png)

# Reproduction log

## **11.15 ~ 11.19： 材料集备（硬件）**

## **11.20 ~ 11.22： 结构设计 + 3D 打印 + 装配**

![FastLivo_machine](https://github.com/user-attachments/assets/55a5412a-9e3b-4a83-9e41-2575b2f2ee87)

## **11.23：STM32代码（发送PWM脉冲）**

## **11.24：Fast-LIO跑通（外接imu）**

## **11.25：外参标定测试**

![FastLivo_ext](https://github.com/user-attachments/assets/6d235952-a04b-47a1-9366-d137048dc46b)

## **11.26 ~ 11.27：相机硬同步 + Lidar硬同步(PPS触发)**

## **11.28 : 软同步（共享内存）**

![image](https://github.com/user-attachments/assets/e73cfc0c-e672-48ba-99f0-6254dc8904d8)
![image](https://github.com/user-attachments/assets/49e0e73f-a432-4131-896c-2c4c046967e3)
时间戳同步了，但是发布时间仍然没有同步，但是保证了没有时延应该就行。
![image](https://github.com/user-attachments/assets/650e31cc-ce40-4284-936e-0ba588563ef9)

## **11.29 : 出图**

![image](https://github.com/user-attachments/assets/c03b83b0-e263-45d5-8bec-0efd5d74a8c6)
![image](https://github.com/user-attachments/assets/57f512bb-dde1-42ee-a207-da8803bb650c)
![image](https://github.com/user-attachments/assets/0c8cc620-9675-41d4-aebd-7e698fef3dbe)
![image](https://github.com/user-attachments/assets/4e86f256-e939-41d7-ac42-d9d3ff2c516a)
![image](https://github.com/user-attachments/assets/d8d4765b-1f53-44a6-a495-69dbda8869bf)

启动顺序：雷达（赋能串口:Stm32->GPS)->相机->IMU（赋能串口sudo chmod 777 /dev/ttyUSB*）->bag包检查时间戳和发布频率->FastLivo

根据Add 0 3D Points，回到源程序，判断可能是点未在方框内，推测是外参矩阵问题，IMU-Lidar的之前在Fast-LIO中试过是可以的。
推测可能是Lidar-Camera的问题，之前标外参是转置了，现在转回来尝试一下，就ok了。
![image](https://github.com/user-attachments/assets/a8bd62b9-804d-4126-bccd-855f9d3f9ee5)
会出现漂移

## **11.30 ： 调优-> 外参标定**

![image](https://github.com/user-attachments/assets/225c5ade-f95b-49aa-af31-235a2eb5aa26)
4个场景，两个室内，两个室外。
4个一起做multi_calib的效果不好
![image](https://github.com/user-attachments/assets/7042e7fb-b2a4-4867-89a7-52b4415cb539)
单独试试，选最优的
![image](https://github.com/user-attachments/assets/413b1d09-d7b7-491c-a1f1-628462070265)
效果不好
![image](https://github.com/user-attachments/assets/a4ae1506-4ec8-4f58-b86c-345aad257f43)
效果不好
![image](https://github.com/user-attachments/assets/87b477d0-3e0c-4644-9cde-ca47f284e0b4)
效果稍微好点
尝试降采样，缩短时间38s->20s
![image](https://github.com/user-attachments/assets/7fcb44dd-7897-49a5-ba5e-a146d75079e7)
效果更差了
![image](https://github.com/user-attachments/assets/b7b25708-8970-4c99-b3c1-422b149b12ff)
效果不好
打算重新标定内参
ROS自带的标定
![image](https://github.com/user-attachments/assets/5a8d026a-6d94-446b-b6f7-26e54b6e7c42)
两次标定的结果相差有点大
![image](https://github.com/user-attachments/assets/f315bfab-063b-4237-a14e-378703e7e71e)

找其他的标定方法

## **12.1~12.2 ：结构设计重构**

![image](https://github.com/user-attachments/assets/245e42ed-99d5-4eea-88f4-d5890433b632)

## **12.3 ： 内外参标定**

内参尝试两套方案：
1、ROS内自带标定
![image](https://github.com/user-attachments/assets/3d9877a4-4718-4d36-91e3-2d169812de22)
2、Kalibr仓库标定
标定焦距方差大
![image](https://github.com/user-attachments/assets/f07b2108-024e-4b0c-a36b-c072a56049e9)

最后使用屏幕作为标定板，解决标定板不平的问题。最后标出来，与出厂数据也较为接近，使用该数据！

![image-20241204144150953](/home/wenming/.config/Typora/typora-user-images/image-20241204144150953.png)

## **12.4 ~ 12.5：IMU时间戳修正**

两个思路：

1. 使用相机内10Hz的IMU + 重新确定IMU-Lidar外参
2. 继续使用WIT的200Hz的IMU + 时间戳偏移（173开始，需偏移到159开始）

第一条线：使用相机内10Hz的IMU + 重新确定IMU-Lidar外参

同样借助火星实验室的lidar_imu_init项目进行标定

![image-20241204202134114](/home/wenming/.config/Typora/typora-user-images/image-20241204202134114.png)

[Final Result] Rotation LiDAR to IMU    =  -8.982750 -84.874467 108.015758 deg
[Final Result] Translation LiDAR to IMU = -0.059569  0.067989 -0.169565 m
[Final Result] Time Lag IMU to LiDAR    = -0.25177276 s 
[Final Result] Bias of Gyroscope        =  0.002945  0.000953 -0.000026 rad/s
[Final Result] Bias of Accelerometer    =  0.040995 -0.039036  0.048515 m/s^2
[Final Result] Gravity in World Frame   = -0.282423 -0.481850 -9.795380 m/s^2

这里Z轴偏移相差16.9cm不太对劲，再标一次

[Final Result] Rotation LiDAR to IMU    = 178.260218 -76.245960 -88.088453 deg
[Final Result] Translation LiDAR to IMU = -0.172350  0.026294  0.022632 m
[Final Result] Time Lag IMU to LiDAR    = -0.16301060 s 
[Final Result] Bias of Gyroscope        =  0.050453 -0.028714 -0.155879 rad/s
[Final Result] Bias of Accelerometer    =  0.016588 -0.016391 -0.069978 m/s^2
[Final Result] Gravity in World Frame   = -0.041471  0.317487 -9.804518 m/s^2

正常不少，这次注意一下让雷达不要怼墙导致退化问题。

[Final Result] Rotation LiDAR to IMU    = 169.596405 -87.405378 -80.441211 deg
[Final Result] Translation LiDAR to IMU = -0.048085 -0.071753 -0.073754 m
[Final Result] Time Lag IMU to LiDAR    = -0.14296537 s 
[Final Result] Bias of Gyroscope        = -0.004753 -0.001572 -0.003281 rad/s
[Final Result] Bias of Accelerometer    = -0.014965 -0.010604 -0.019126 m/s^2
[Final Result] Gravity in World Frame   = -0.626495 -0.249837 -9.788097 m/s^2

具体旋转矩阵看他保存的路径即可

差不多，并且根据159和173的时间差，确实是相差14s左右

![image-20241204203901240](/home/wenming/.config/Typora/typora-user-images/image-20241204203901240.png)

![image-20241204205214269](/home/wenming/.config/Typora/typora-user-images/image-20241204205214269.png)

 0.007541 -0.999859 -0.015027 -0.048085
-0.044751  0.014675 -0.998890 -0.071753
 0.998970  0.008205 -0.044634 -0.073754
 0.000000  0.000000  0.000000  1.000000

开始运行Fast-LIVO

![image-20241204210435820](/home/wenming/.config/Typora/typora-user-images/image-20241204210435820.png)

还是会出现漂移

![image-20241204210801258](/home/wenming/.config/Typora/typora-user-images/image-20241204210801258.png) 

时间戳为一致的，但是不确保为无时延系统

考虑是距离太近的原因

![image-20241204211039404](/home/wenming/.config/Typora/typora-user-images/image-20241204211039404.png)

调远距离，仍然跑飞 

第二条线：继续使用WIT的200Hz的IMU+ 时间戳偏移（173开始，需偏移到159开始）

以捕获到IMU和雷达第一帧的时间为准？做个时间差，尝试修改源代码laserMapping.cpp

修改了imu_cbk里的代码

![image-20241205114401147](/home/wenming/.config/Typora/typora-user-images/image-20241205114401147.png)

修改了img_cbk里的代码

![image-20241205114458498](/home/wenming/.config/Typora/typora-user-images/image-20241205114458498.png)

不动的时候能够正常出图

![image-20241205114522764](/home/wenming/.config/Typora/typora-user-images/image-20241205114522764.png)

但是移动仍然漂移严重

![image-20241205114706329](/home/wenming/.config/Typora/typora-user-images/image-20241205114706329.png)

![image-20241205114749831](/home/wenming/.config/Typora/typora-user-images/image-20241205114749831.png)

相机换了新内参后的外参标定

![image-20241205125452508](/home/wenming/.config/Typora/typora-user-images/image-20241205125452508.png)

0.00874713,-0.999851,-0.0149063,-0.234126
-0.0265367,0.0146695,-0.99954,-0.0372584
0.99961,0.00913867,-0.0264044,0.0715121
0,0,0,1

修改曝光时间为10ms,50ms都是会飞，用相机的imu就需要IMU初始化，用wit的imu就不需要IMU初始化

![image-20241205161557481](/home/wenming/.config/Typora/typora-user-images/image-20241205161557481.png)

## **12.6 : 装配新硬件 + 找到漂移原因：时间戳精确度问题**

相机使用的是等待infra1，infra2，depth，所有流一起到达的时间，延时高，尝试修改。

1. 删除了除了color和imu流的其他流，加速通信速度。

2. 不使用同步功能（需要等待，会影响捕获时间的精度）

虽然相机还是30Hz，IMU为200Hz，但是时间精准，然后在fastlivo源代码接收imu和img的地方做个delta time的时间偏移，将雷达和相机时间统一到一个时间轴上，就可以不漂移了。硬同步软同步都没有做。但是在雷达退化的时候会产生漂移现象。

但是delta_time的计算仍然有bug，放到一个launch文件里启动？

## **12.7 ：修改delta_time计算bug + 低频相机(硬同步 + 软同步) + 雷达1->10拆包查看**

**修改delta_time计算bug：**

bug已修复，由于image的频率更高，所以概率上image比lidar更早到，当img先到的时候，lidar还没到，那么lidar_init_time还没被赋值，仍为0，此时就会出现问题。加上一个限制，只有雷达到了，才能进行delta_time的计算。

![image-20241207124421236](/home/wenming/.config/Typora/typora-user-images/image-20241207124421236.png)

![image-20241207124315163](/home/wenming/.config/Typora/typora-user-images/image-20241207124315163.png)

![image-20241207124346810](/home/wenming/.config/Typora/typora-user-images/image-20241207124346810.png)

**修复相机无法一次ctrl+c及时退出的bug：**

![image-20241207150427132](/home/wenming/.config/Typora/typora-user-images/image-20241207150427132.png)

重新设置为cleand4345i.cpp

**低频相机(硬同步 + 软同步) ：**

尝试使用能控的Stereo Module来做一个硬同步,但是需要重新标定内参和外参。

尝试精简源代码，看是否能使color_image的时延降低！

尝试1：使用了精简版低频的color + ros::Time::Now来做，会漂移 roslaunch hwsync d435i_controlcolor.launch 

尝试2：使用了精简版低频的color + color_frame.get_timestamp()来做，时间戳为214开头，初始化完后，直接飞天？

![image-20241207221526731](/home/wenming/.config/Typora/typora-user-images/image-20241207221526731.png)

![image-20241207221534901](/home/wenming/.config/Typora/typora-user-images/image-20241207221534901.png)

并且会报这个错误，尝试增大了config里的参数，但是仍然不行。



**雷达1->10拆包查看：**

![image-20241207142146682](/home/wenming/.config/Typora/typora-user-images/image-20241207142146682.png)

播放包，用rviz查看，是不断旋转的，可以确定是扫到一部分就发一部分的。

![image-20241207144630137](/home/wenming/.config/Typora/typora-user-images/image-20241207144630137.png)



**录包：hit_indoor.bag 10G**

![image-20241207154343634](/home/wenming/.config/Typora/typora-user-images/image-20241207154343634.png)

快速移动，雷达退化的时候都会导致漂移

尝试官方demo，修改了lasermapping后，导致官方demo都出现了漂移

![image-20241207171947612](/home/wenming/.config/Typora/typora-user-images/image-20241207171947612.png)

## **12.8 ：CSDN思路 **

尝试1：https://blog.csdn.net/weixin_40599145/article/details/128064323

修改livox驱动，修改camera驱动，然后启动，无硬同步，软同步。创建一个ky_ws，进行尝试

![image-20241208133352741](/home/wenming/.config/Typora/typora-user-images/image-20241208133352741.png)

大的晃动还是会跑飞,这个好像还是加了GPRMC的，换回原版的livox_ros_driver试试

![image-20241208135242542](/home/wenming/.config/Typora/typora-user-images/image-20241208135242542.png)

原地还是会抖动

## **12.9 ：简书思路 + 系统时间（纪年法）尝试**

尝试1：https://www.jianshu.com/p/d4b5cf3c1475



尝试2：系统时间尝试，仍然沿用hwsync代码



## **12.10：尝试硬件时间**

freecc

![image-20241210102012542](/home/wenming/.config/Typora/typora-user-images/image-20241210102012542.png)

hwtime_d435i 硬件时间

![image-20241210102453477](/home/wenming/.config/Typora/typora-user-images/image-20241210102453477.png)

可能imu也需要使用硬件时间？

![image-20241210104657690](/home/wenming/.config/Typora/typora-user-images/image-20241210104657690.png)

## **12.11 海康d435i外参标定、imu标定、海康相机同步、简书思路、尝试多种时间戳**

### 海康相机和雷达共同使用：配置不同网段网口

- 在设置中添加配置

<img src="/home/wenming/.config/Typora/typora-user-images/image-20241211115925091.png" alt="image-20241211115925091" style="zoom:50%;" />

海康相机ip设置

- 先设置ubuntu中网口地址
  - <img src="/home/wenming/.config/Typora/typora-user-images/image-20241211115711237.png" alt="image-20241211115711237" style="zoom:50%;" />

- 再在MVS客户端中右键修改IP地址
  - <img src="/home/wenming/.config/Typora/typora-user-images/image-20241211115826651.png" alt="image-20241211115826651" style="zoom:50%;" />
- 雷达保持不变

注意：网段不能一样192.168.1.XX 和 192.168.2.XX



### 海康d435i外参标定

总是配不准

![image-20241211200940703](/home/wenming/.config/Typora/typora-user-images/image-20241211200940703.png)

发现雷达的点云漂移严重

![image-20241211200618851](/home/wenming/.config/Typora/typora-user-images/image-20241211200618851.png)

剔除这一束外点后

![image-20241211201225194](/home/wenming/.config/Typora/typora-user-images/image-20241211201225194.png)

注意不能转成ply后再转回pcd，会丢失intensity信息

重新标注外参

![image-20241211201433270](/home/wenming/.config/Typora/typora-user-images/image-20241211201433270.png)

还是不对劲，，可能是照片的问题，左上角的阳光？

扫描地板的时候出现了负高度的情况，

![image-20241211215358112](/home/wenming/.config/Typora/typora-user-images/image-20241211215358112.png)

可能是地板的反光问题，找了一处毛坯地板at茶水间，检查点云没问题

![image-20241211214603720](/home/wenming/.config/Typora/typora-user-images/image-20241211214603720.png)

<img src="/home/wenming/.config/Typora/typora-user-images/image-20241211214528913.png" alt="image-20241211214528913" style="zoom:67%;" />

先尝试d435i的标定

![image-20241211214847386](/home/wenming/.config/Typora/typora-user-images/image-20241211214847386.png)

![image-20241211214835750](/home/wenming/.config/Typora/typora-user-images/image-20241211214835750.png)

效果还是不行

改成indoor.yaml试一下

![image-20241211215223756](/home/wenming/.config/Typora/typora-user-images/image-20241211215223756.png)

![image-20241211215114622](/home/wenming/.config/Typora/typora-user-images/image-20241211215114622.png)

效果还不错

继续标定海康相机的外参

![image-20241211225600076](/home/wenming/.config/Typora/typora-user-images/image-20241211225600076.png)

效果不好，看看别人的像素比例如何gitee —— https://gitee.com/gwmunan/fast-livo_-reproduction

## 12.12  海康相机外参标定 

![image-20241212200934882](/home/wenming/.config/Typora/typora-user-images/image-20241212200934882.png)

模仿yaml文件配置自己的相机

外参标定始终无法成功

## 12.13~12.14 调海康相机驱动 + 外参标定

 使用ros驱动开启1280 x 1024 的像素有横条问题

使用lidar_camera标定始终无法成功

卡住了

## 12.15~12.16 海康相机内外参标定

使用gundasmart代码并debug调通，外部触发线连接，客户端内配置为线路0触发

使用ros驱动开启 1280 x 1024 的像素，始终会有横条问题出现，后更换至3072*2048正常

由于lidar_d435i是ok的，故打算使用kalibr的多相机标定进行外参标定->可得到hik相机_lidar之间的外参

