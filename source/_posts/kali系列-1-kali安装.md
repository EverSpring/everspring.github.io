---
title: kali系列-一.kali U盘安装
date: 2022-04-30 21:02:28
author: EverSpring
toc: false
mathjax: false
categories: kali
tags:
  - 渗透测试
  - kali U盘安装
---
kali系统是渗透测试的瑞士军刀，集成了大部分常用的Hack工具。
说说U盘安装kali的好处：

1. 虚拟机安装kali，不支持wifi，如果要支持wifi需要额外使用无线网卡
2. 不用虚拟机，要么装双系统、要么再准备一台电脑（土豪随意）
3. 装电脑里不方便携带

### 一、前期准备
* U盘
  建议USB 3.0，2.0比较卡。
  至少16G，建议32G以上
* DiskGenius U盘分区工具
  百度找
* Universal-USB-Installer iso系统镜像烧录工具
  下载地址：http://downza.51speed.top/2022/02/08/UniversalUSB.rar?timestamp=626cdc34&auth_key=332de8aadceebdf3002d5c208110e680  
  推荐使用，原因：小巧，只有2M左右。
  不推荐UltraISO，我这烧录后有问题
* kali live iso镜像下载
我的是2021.2-live-amd64.iso
镜像地址，国外的下载慢，建议用国内的。
清华大学镜像：https://mirrors.tuna.tsinghua.edu.cn/kali-images/
中国科学技术大学：https://mirrors.ustc.edu.cn/kali-images/

### 二、使用
* **1. DiskGenius分区**
  <img src="https://raw.githubusercontent.com/EverSpring/picbed/main/202205082341257.png" alt="图1" style="zoom: 50%;" />
  <img src="https://raw.githubusercontent.com/EverSpring/picbed/main/202205082341891.png" alt="图2" style="zoom: 50%;" />
* <img src="https://raw.githubusercontent.com/EverSpring/picbed/main/202205082342164.png" alt="image-20220430235104169" style="zoom:50%;" />
  <img src="https://raw.githubusercontent.com/EverSpring/picbed/main/202205082342039.png" alt="image-20220430235741924" style="zoom:50%;" />
  <img src="https://raw.githubusercontent.com/EverSpring/picbed/main/202205082342428.png" alt="image-20220430235926700" style="zoom:50%;" />
* **2. Universal-USB-Installer烧制启动盘**

<img src="https://raw.githubusercontent.com/EverSpring/picbed/main/202205082342833.png" style="zoom: 80%;" />

完成后得到了一个可以u盘启动的kali 操作系统，即kali live usb

* **3. U盘启动**
  在第2步烧制后，重启电脑，进入BIOS系统。进入BIOS系统各品牌的方式，见https://jingyan.baidu.com/article/cbf0e500b4c71b2eaa2893fd.html

  <img src="https://raw.githubusercontent.com/EverSpring/picbed/main/202205082342387.png" alt="image-20220501000544177" style="zoom: 67%;" />

进入系统后这两种都可以。下面一种是自动支持持久化的，但持久化的是文件，系统配置持久化暂时还没找到方法
> kali 2019后系统的用户是kali，密码也是kali。
> 系统没有提供root用户，但提供了sudo权限，所以后续一些提示权限不足的，就用sudo xxx执行即可