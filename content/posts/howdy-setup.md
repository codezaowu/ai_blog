+++
title = 'Howdy 人脸识别安装与排坑指南'
date = 2026-06-15T21:00:00+08:00
draft = false
description = '在 Ubuntu 24.04 上安装 Howdy 人脸识别认证的完整记录，包含踩坑修复'
tags = ['howdy', 'ubuntu', 'face-recognition', 'pam']
+++

## 前言

Howdy 是 Linux 下类似 Windows Hello 的人脸识别认证工具。本文记录在 Ubuntu 24.04 上的完整安装过程，包括踩过的坑和修复方案。

## 安装

### 1. 添加 PPA

```bash
sudo add-apt-repository ppa:boltgolt/howdy
sudo apt update
```

### 2. 安装系统依赖

```bash
sudo apt install -y python3-numpy python3-opencv v4l-utils \
    build-essential cmake libopenblas-dev python3-dev python3-pip
```

### 3. 安装 howdy

```bash
sudo apt install howdy
```

### 4. 安装 dlib

Python 3.12 的 `EXTERNALLY-MANAGED` 会阻止 pip，需要临时解除：

```bash
sudo mv /usr/lib/python3.12/EXTERNALLY-MANAGED \
        /usr/lib/python3.12/EXTERNALLY-MANAGED.bak
sudo pip3 install dlib --no-build-isolation
sudo mv /usr/lib/python3.12/EXTERNALLY-MANAGED.bak \
        /usr/lib/python3.12/EXTERNALLY-MANAGED
```

## 踩坑记录

### 坑 1: Python 解释器路径错误

系统有两个 Python：Miniconda 3.13（缺少 dlib/opencv）和系统 3.12（完整）。`#!/usr/bin/env python3` 捡到了 miniconda 的。

修复：

```bash
sudo sed -i '1s|#!/usr/bin/env python3|#!/usr/bin/python3|' \
    /lib/security/howdy/cli.py
```

### 坑 2: pam.py 是 Python 2 语法

`pam_python.so` 实际链接 Python 3.12，但代码用了 `import ConfigParser`（Python 2）。

修复：

```bash
sudo sed -i 's/import ConfigParser/import configparser/;
            s/ConfigParser.ConfigParser()/configparser.ConfigParser()/' \
    /lib/security/howdy/pam.py
```

### 坑 3: 快照目录不可写

人脸识别后生成快照时目录不存在，导致 PAM 认为认证失败。

修复：

```bash
sudo mkdir -p /usr/lib/security/howdy/snapshots
sudo chmod 777 /usr/lib/security/howdy/snapshots
```

### 坑 4: PAM 未激活

```bash
sudo DEBIAN_FRONTEND=noninteractive pam-auth-update --package howdy
```

### 坑 5: 符号链接未创建

```bash
sudo ln -sf /lib/security/howdy/cli.py /usr/local/bin/howdy
```

## 最终效果

```bash
# 添加人脸
sudo howdy -U john add

# 测试
sudo howdy list
sudo -k && sudo echo "face auth ok"
```

现在 sudo、锁屏等都会先尝试人脸识别，超时或失败后降级到密码。
