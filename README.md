# livox mapping

![image-20241209161847378](assets/image-20241209161847378.png)

![image-20241209161902542](assets/image-20241209161902542.png)

# Reproduction log

## **11.15 ~ 11.19： 材料集备（硬件）**

## **11.20 ~ 11.22： 结构设计 + 3D 打印 + 装配**

![FastLivo_machine](https://github.com/user-attachments/assets/55a5412a-9e3b-4a83-9e41-2575b2f2ee87)

## **11.23：STM32代码（发送PWM脉冲）**

## **11.24：Fast-LIO跑通（外接imu）**

![image-20241226203923339](assets/image-20241226203923339.png)

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

![image-20241216113614740](assets/image-20241216113614740.png)

## **12.4 ~ 12.5：IMU时间戳修正**

两个思路：

1. 使用相机内10Hz的IMU + 重新确定IMU-Lidar外参
2. 继续使用WIT的200Hz的IMU + 时间戳偏移（173开始，需偏移到159开始）

第一条线：使用相机内10Hz的IMU + 重新确定IMU-Lidar外参

同样借助火星实验室的lidar_imu_init项目进行标定

![image-20241216113626274](assets/image-20241216113626274.png)

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

![image-20241216114044441](assets/image-20241216114044441.png)

![image-20241204205214269](assets/image-20241204205214269.png)

 0.007541 -0.999859 -0.015027 -0.048085
-0.044751  0.014675 -0.998890 -0.071753
 0.998970  0.008205 -0.044634 -0.073754
 0.000000  0.000000  0.000000  1.000000

开始运行Fast-LIVO

![image-20241204210435820](assets/image-20241204210435820.png)

还是会出现漂移

![image-20241204210801258](assets/image-20241204210801258.png) 

时间戳为一致的，但是不确保为无时延系统

考虑是距离太近的原因

![image-20241204211039404](assets/image-20241204211039404.png)

调远距离，仍然跑飞 

第二条线：继续使用WIT的200Hz的IMU+ 时间戳偏移（173开始，需偏移到159开始）

以捕获到IMU和雷达第一帧的时间为准？做个时间差，尝试修改源代码laserMapping.cpp

修改了imu_cbk里的代码

![image-20241205114401147](assets/image-20241205114401147.png)

修改了img_cbk里的代码

![image-20241205114458498](assets/image-20241205114458498.png)

不动的时候能够正常出图

![image-20241205114522764](assets/image-20241205114522764.png)

但是移动仍然漂移严重

![image-20241205114706329](assets/image-20241205114706329.png)

![image-20241205114749831](assets/image-20241205114749831.png)

相机换了新内参后的外参标定

![image-20241205125452508](assets/image-20241205125452508.png)

0.00874713,-0.999851,-0.0149063,-0.234126
-0.0265367,0.0146695,-0.99954,-0.0372584
0.99961,0.00913867,-0.0264044,0.0715121
0,0,0,1

修改曝光时间为10ms,50ms都是会飞，用相机的imu就需要IMU初始化，用wit的imu就不需要IMU初始化

![image-20241205161557481](assets/image-20241205161557481.png)

## **12.6 : 装配新硬件 + 找到漂移原因：时间戳精确度问题**

相机使用的是等待infra1，infra2，depth，所有流一起到达的时间，延时高，尝试修改。

1. 删除了除了color和imu流的其他流，加速通信速度。

2. 不使用同步功能（需要等待，会影响捕获时间的精度）

虽然相机还是30Hz，IMU为200Hz，但是时间精准，然后在fastlivo源代码接收imu和img的地方做个delta time的时间偏移，将雷达和相机时间统一到一个时间轴上，就可以不漂移了。硬同步软同步都没有做。但是在雷达退化的时候会产生漂移现象。

但是delta_time的计算仍然有bug，放到一个launch文件里启动？

## **12.7 ：修改delta_time计算bug + 低频相机(硬同步 + 软同步) + 雷达1->10拆包查看**

**修改delta_time计算bug：**

bug已修复，由于image的频率更高，所以概率上image比lidar更早到，当img先到的时候，lidar还没到，那么lidar_init_time还没被赋值，仍为0，此时就会出现问题。加上一个限制，只有雷达到了，才能进行delta_time的计算。

![image-20241207124421236](assets/image-20241207124421236.png)

![image-20241207124315163](assets/image-20241207124315163.png)

![image-20241207124346810](assets/image-20241207124346810.png)

**修复相机无法一次ctrl+c及时退出的bug：**

![image-20241207150427132](assets/image-20241207150427132.png)

重新设置为cleand4345i.cpp

**低频相机(硬同步 + 软同步) ：**

尝试使用能控的Stereo Module来做一个硬同步,但是需要重新标定内参和外参。

尝试精简源代码，看是否能使color_image的时延降低！

尝试1：使用了精简版低频的color + ros::Time::Now来做，会漂移 roslaunch hwsync d435i_controlcolor.launch 

尝试2：使用了精简版低频的color + color_frame.get_timestamp()来做，时间戳为214开头，初始化完后，直接飞天？

![image-20241207221526731](assets/image-20241207221526731.png)

![image-20241207221534901](assets/image-20241207221534901.png)

并且会报这个错误，尝试增大了config里的参数，但是仍然不行。



**雷达1->10拆包查看：**

![image-20241207142146682](assets/image-20241207142146682.png)

播放包，用rviz查看，是不断旋转的，可以确定是扫到一部分就发一部分的。

![image-20241207144630137](assets/image-20241207144630137.png)



**录包：hit_indoor.bag 10G**

![image-20241207154343634](assets/image-20241207154343634.png)

快速移动，雷达退化的时候都会导致漂移

尝试官方demo，修改了lasermapping后，导致官方demo都出现了漂移

![image-20241207171947612](assets/image-20241207171947612.png)

## **12.8 ：CSDN思路 **

尝试1：https://blog.csdn.net/weixin_40599145/article/details/128064323

修改livox驱动，修改camera驱动，然后启动，无硬同步，软同步。创建一个ky_ws，进行尝试

![image-20241208133352741](assets/image-20241208133352741.png)

大的晃动还是会跑飞,这个好像还是加了GPRMC的，换回原版的livox_ros_driver试试

![image-20241208135242542](assets/image-20241208135242542.png)

原地还是会抖动

## **12.9 ：简书思路 + 系统时间（纪年法）尝试**

尝试1：https://www.jianshu.com/p/d4b5cf3c1475



尝试2：系统时间尝试，仍然沿用hwsync代码



## **12.10：尝试硬件时间**

freecc

![image-20241210102012542](assets/image-20241210102012542.png)

hwtime_d435i 硬件时间

![image-20241210102453477](assets/image-20241210102453477.png)

可能imu也需要使用硬件时间？

![image-20241210104657690](assets/image-20241210104657690.png)

## **12.11 海康d435i外参标定、imu标定、海康相机同步、简书思路、尝试多种时间戳**

### 海康相机和雷达共同使用：配置不同网段网口

- 在设置中添加配置

<img src="assets/image-20241211115925091.png" alt="image-20241211115925091" style="zoom:50%;" />

海康相机ip设置

- 先设置ubuntu中网口地址
  - <img src="assets/image-20241211115711237.png" alt="image-20241211115711237" style="zoom:50%;" />

- 再在MVS客户端中右键修改IP地址
  - <img src="assets/image-20241211115826651.png" alt="image-20241211115826651" style="zoom:50%;" />
- 雷达保持不变

注意：网段不能一样192.168.1.XX 和 192.168.2.XX



### 海康d435i外参标定

总是配不准

![image-20241211200940703](assets/image-20241211200940703.png)

发现雷达的点云漂移严重

![image-20241211200618851](assets/image-20241211200618851.png)

剔除这一束外点后

![image-20241211201225194](assets/image-20241211201225194.png)

注意不能转成ply后再转回pcd，会丢失intensity信息

重新标注外参

![image-20241211201433270](assets/image-20241211201433270.png)

还是不对劲，，可能是照片的问题，左上角的阳光？

扫描地板的时候出现了负高度的情况，

![image-20241211215358112](assets/image-20241211215358112.png)

可能是地板的反光问题，找了一处毛坯地板at茶水间，检查点云没问题

![image-20241211214603720](assets/image-20241211214603720.png)

<img src="assets/image-20241211214528913.png" alt="image-20241211214528913" style="zoom:67%;" />

先尝试d435i的标定

![image-20241211214847386](assets/image-20241211214847386.png)

![image-20241211214835750](assets/image-20241211214835750.png)

效果还是不行

改成indoor.yaml试一下

![image-20241211215223756](assets/image-20241211215223756.png)

![image-20241211215114622](assets/image-20241211215114622.png)

效果还不错

继续标定海康相机的外参

![image-20241216113436999](assets/image-20241216113436999.png)

 [效果不好，看看别人的像素比例如何gitee](https://gitee.com/gwmunan/fast-livo_-reproduction)

## 12.12  海康相机外参标定 

模仿gitee的yaml文件配置自己的相机

外参标定始终无法成功，可能是横条问题

## 12.13~12.14 调海康相机驱动 + 外参标定

 使用ros驱动开启1280 x 1024 的像素有横条问题

使用lidar_camera标定始终无法成功

卡住了

## 12.15~12.16 海康相机内外参标定

使用gundasmart代码并debug调通，外部触发线连接，客户端内配置为线路0触发

使用ros驱动开启 1280 x 1024 的像素，始终会有横条问题出现，后更换至3072*2048正常

由于lidar_d435i是ok的，故打算使用kalibr的多相机标定进行外参标定->可得到hik相机_lidar之间的外参

```
rosrun kalibr kalibr_calibrate_cameras --models pinhole-radtan pinhole-radtan --target aprilgrid_kyipad.yaml --bag ~/Desktop/catkin_ws/2024-12-15-21-44-57.bag --topics /hikrobot_camera/rgb /camera/color/image_raw --show-extraction --approx-sync 0.04
```

调通github联动contributor



## 12.25-12.26 继续尝试标定

进行了多次的户外标定后，均已失败告终。尝试室内标定。

![image-20241226212314554](assets/image-20241226212314554.png)

![image-20241226212437760](assets/image-20241226212437760.png)

重新imu-lidar标定，使用FastLIO里的Lidar_Imu_Init包

![image-20241226212500031](assets/image-20241226212500031.png)

<img src="assets/image-20241226212827031.png" alt="image-20241226212827031" style="zoom: 50%;" />

和之前尝试的FastLivo轨迹相同，但是此次没有出现漂移，所以关键还是相机带来的可能进行了负优化。所以关键点还是外参和时间戳的对齐。

![image-20241226212730344](assets/image-20241226212730344.png)

尝试原始雷达、IMU驱动 + FastLivo里的deltatime处理:

![image-20241226215752836](assets/image-20241226215752836.png)

出现跑不动的情况，原因是3072*2048的图像处理起来太耗时了，故重新标定了1280\*1024，如下所示：

![image-20241227110029456](assets/image-20241227110029456.png)

出图如下所示：

![image-20241226223722565](assets/image-20241226223722565.png)

![image-20241226223741135](assets/image-20241226223741135.png)

![image-20241226223753461](assets/image-20241226223753461.png)

接下来有用Lidar_Imu_Init跑了一下，做了极限测试，看看上限，表现得很好。如下所示为机械楼。

![image-20241226224531404](assets/image-20241226224531404.png)

![image-20241226224612311](assets/image-20241226224612311.png)

![image-20241226224648511](assets/image-20241226224648511.png)

![image-20241226224703811](assets/image-20241226224703811.png)

![image-20241226224729393](assets/image-20241226224729393.png)

![image-20241226224808400](assets/image-20241226224808400.png)

![image-20241226224904221](assets/image-20241226224904221.png)

![image-20241226224953983](assets/image-20241226224953983.png) 	![image-20241226225008926](assets/image-20241226225008926.png)

## 12.27 尝试时间同步

思路1：雷达驱动的四个方案——LIV_HandHold、vell001、CSDN(D435i+Avia)、简达智能

思路2：是否可以学FastLio（Lidar_Imu_Init）里找到完美的Lidar_Imu_deltatime 和 Lidar_Camera_deltatime















# 标定

[参考链接joint-lidar-camera-calib](https://github.com/hku-mars/joint-lidar-camera-calib)

## 使用

### 数据采集

1. 多纹理平面，至少需要三个具有非共面法向量的平面向量的平面。否则，场景会导致点到平面配准的退化，从而损害外参校准精度。
2. 初始外参，给定初始外在参数（初始旋转误差< 5度，初始平移误差< 0.5 m），6~8帧数据通常足以校准参数。用户在记录新帧时稍微改变传感器套件的偏航角（约 5 度）和 xy 平移（约 10 厘米）。请注意，**应避免纯平移（无旋转）**，因为这会导致相机自校准的退化。如果能够解决上述问题，连续运动也是可以接受的。
3. 如果外部参数的初始猜测不可用，用户可以使用手眼校准来恢复它们。提供了示例代码（ *src/hand_eye_calib.cpp* ）和位姿文件（ *sample_data/hand_eye_calib* ）。

### 初始化

1. 相机自校准

2. 激光雷达位姿估计

   - 首先，使用增量点到平面配准来估计每个 LiDAR 位姿（在*config/registration.yaml*中输入您的数据路径）：

     ```
     roslaunch balm2 registration.launch
     ```

   - 进行 LiDAR  BA调整（在*config/conduct_BA.yaml*中输入您的数据路径）：

     ```
     roslaunch balm2 conduct_BA.launch
     ```

3. 联合标定，按如下方式组织数据文件夹：

   ```
   .
   ├── clouds
   │   ├── 0.pcd
   │   ├── ...
   │   └── x.pcd
   ├── config
   │   └── config.yaml
   ├── images
   │   ├── 0.png
   │   ├── ...
   │   └── x.png
   ├── LiDAR_pose
   │   └── lidar_poses_BA.txt
   ├── result
   └── SfM
       ├── cameras.txt
       ├── images.txt
       └── points3D.txt
   ```

   联合优化

   ```
   roslaunch joint_lidar_camera_calib calib.launch
   ```

### 适应性

1. 仅外部校准，请在*config/config.yaml*中将*keep_intrinsic_fixed*设置为*true* 























