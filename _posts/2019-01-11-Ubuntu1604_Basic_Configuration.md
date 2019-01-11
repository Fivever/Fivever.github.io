---
published: true
layout: post
title: Ubuntu1604 Basic Configuration
tags:
  - Data Structure
description: 基于Ubuntu1604的基础开发配置
toc: true
share: true
comments: true

---
### 更改系统软件下载源
```
cd /etc/apt
sudo cp sources.list sources.list.bak
sudo gedit sources.list

deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial universe
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates universe
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security universe
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security multiverse
```
### 更新软件
```
sudo apt-get update
sudo apt-get upgrade
```
### 卸载不必要的软件
```
sudo apt-get remove libreoffice-common
sudo apt-get remove unity-webapps-common
sudo apt-get remove thunderbird totem rhythmbox empathy brasero simple-scan gnome-mahjongg aisleriot
sudo apt-get remove gnome-mines cheese transmission-common gnome-orca webbrowser-app gnome-sudoku landscape-client-ui-install
sudo apt-get remove onboard deja-dup 
```
### 安装vim
```
sudo apt-get remove vim-common
sudo apt-get install vim
```
### 安装miniconda3
```
cd Downloads
bash Miniconda3-latest-Linux-x86_64.sh
source ~/.bashrc
```
### 安装C++编译环境
```
sudo apt-get install build-essential
```
### 安装java编译环境
```
sudo apt-get install python-software-properties
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
sudo apt install oracle-java8-set-default
```
### 更改下载源并安装pytorch
```
conda config --prepend channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda install pytorch-cpu torchvision-cpu -c pytorch
```
### 使支持exfat文件格式
```
sudo apt-get install exfat-fuse
```
