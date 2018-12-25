# mac os安装rz和sz的方法

[TOC]

## 前提

开发人员在开发时有时会遇到需要在本机和开发机之间传文件的问题。虽然Mac下的scp命令可以完成文件的上传和下载功能，但如果开发机的登陆需要经过跳板机时，scp命令就没有办法正常使用了。

下面介绍一下Mac OS下在iTerm2如何配置rz，sz。（sz 是send，表示从服务器发送文件到本机。rz 是receive，表示服务器接收来自本机的文件。）

## 步骤

### 1.安装lrzsz

lrzsz是一款在linux里可代替ftp上传和下载的程序。通过下载它来使用rz，sz。

```java
brew install lrzsz
```

### 2.安装wget

下载lrzsz之后，需要使用`wget`下载iterm2-zmodem。Mac默认不安装wget，可以通过brew安装。

```java
brew install wget
```

### 3.下载iterm2-zmodem

在iTerm2中使用Zmodem传输文件。

```java
cd /usr/local/bin

wget https://raw.github.com/mmastrac/iterm2-zmodem/master/iterm2-send-zmodem.sh

wget https://raw.github.com/mmastrac/iterm2-zmodem/master/iterm2-recv-zmodem.sh

chmod 777 /usr/local/bin/iterm2-*

```

### 4.在iterm2的preferences中依次选择：

>  Profiles --> [Your Profile] --> Advanced --> Triggers --> Edit

如图：

![image-20181114002706475](https://ws2.sinaimg.cn/large/006tNbRwly1fx6wvvrqqdj31jy10s0zv.jpg)

### 5.在Triggers选项中，新建两个triggers

- rz命令的trigger：

```java
Regular expression: rz waiting to receive.\*\*B0100
 Action: Run Silent Coprocess
 Parameters: /usr/local/bin/iterm2-send-zmodem.sh
 Instant: checked
```

- sz命令的trigger：

```java
Regular expression: \*\*B00000000000000
Action: Run Silent Coprocess
Parameters: /usr/local/bin/iterm2-recv-zmodem.sh
Instant: checked
```

如下图：

![image-20181114003131568](https://ws3.sinaimg.cn/large/006tNbRwly1fx6ww44uxaj31jw0qq7bh.jpg)



### 6.重启iterm2，连接远程Linux，输入rz命令尝试一下

























