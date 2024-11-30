# Reproduction log
**11.15 ~ 11.19： 材料集备（硬件）**

**11.20 ~ 11.22： 结构设计 + 3D 打印 + 装配**
![FastLivo_machine](https://github.com/user-attachments/assets/55a5412a-9e3b-4a83-9e41-2575b2f2ee87)

**11.23：STM32代码（发送PWM脉冲）**

**11.24：Fast-LIO跑通（外接imu）**

**11.25：外参标定测试**
![FastLivo_ext](https://github.com/user-attachments/assets/6d235952-a04b-47a1-9366-d137048dc46b)

**11.26 ~ 11.27：相机硬同步 + Lidar硬同步(PPS触发)**

**11.28 : 软同步（共享内存）**
![image](https://github.com/user-attachments/assets/e73cfc0c-e672-48ba-99f0-6254dc8904d8)
![image](https://github.com/user-attachments/assets/49e0e73f-a432-4131-896c-2c4c046967e3)
时间戳同步了，但是发布时间仍然没有同步，但是保证了没有时延应该就行。
![image](https://github.com/user-attachments/assets/650e31cc-ce40-4284-936e-0ba588563ef9)


**11.29 : 出图**
根据Add 0 3D Points，回到源程序，判断可能是点未在方框内，推测是外参矩阵问题，IMU-Lidar的之前在Fast-LIO中试过是可以的。
推测可能是Lidar-Camera的问题，之前标外参是转置了，现在转回来尝试一下，就ok了。
![image](https://github.com/user-attachments/assets/a8bd62b9-804d-4126-bccd-855f9d3f9ee5)
会出现漂移

**11.30 ： 调优-> 外参标定**
![image](https://github.com/user-attachments/assets/225c5ade-f95b-49aa-af31-235a2eb5aa26)
4个场景，两个室内，两个室外。
4个一起做multi_calib的效果不好
![image](https://github.com/user-attachments/assets/7042e7fb-b2a4-4867-89a7-52b4415cb539)
单独试试，选最优的
![image](https://github.com/user-attachments/assets/413b1d09-d7b7-491c-a1f1-628462070265)
效果不好
![image](https://github.com/user-attachments/assets/a4ae1506-4ec8-4f58-b86c-345aad257f43)
效果不好
