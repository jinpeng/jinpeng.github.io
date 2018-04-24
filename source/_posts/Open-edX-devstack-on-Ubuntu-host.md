title: Open edX devstack on Ubuntu host
date: 2016-04-21 17:47:25
tags: open edx devstack ubuntu
---
      
# Open edX Devstack Ubuntu host #

安装环境，Ubuntu 14.04 LTS Desktop，ThinkPad T450.

## 安装VirtualBox和Vagrant ##

```bash
$ sudo vi /etc/apt/sources.list
```
在文档最后添加VirtualBox源：
```
deb http://download.virtualbox.org/virtualbox/debian trusty contrib
```


```bash
$ wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | sudo apt-key add -
$ sudo apt-get update
$ sudo apt-get install virtualbox-5.0
```

```bash
$ wget https://releases.hashicorp.com/vagrant/1.8.1/vagrant_1.8.1_x86_64.deb
$ sudo dpkg -i vagrant_1.8.1_x86_64.deb
```

安装Vagrant plugins，vagrant-vbguest等。  
修改Vagrant所用Ruby gems服务器为淘宝的镜像： 

```bash
$ grep -rl "https://rubygems.org" /opt/vagrant/embedded/gems/ | xargs sudo sed -i 's/https\:\/\/rubygems.org/https\:\/\/ruby.taobao.org/g'
$ grep -rl "https://rubygems.org" /opt/vagrant/embedded/gems/
$ vagrant plugin install vagrant-vbguest
```


## 导入provision好的devstack box ##

安装配置sftp server，用于传输文件，也可以用其它方式。

```bash
$ sudo apt-get install openssh-server
$ sudo addgroup sftp-users
$ sudo adduser alice
$ sudo usermod -G sftp-users -s /bin/false alice
$ sudo addgroup ssh-users
$ sudo usermod -a -G ssh-users jinpeng
$ sudo mkdir /home/sftp_root
$ sudo mkdir /home/sftp_root/shared
$ sudo chown jinpeng:sftp-users /home/sftp_root/shared
$ sudo chmod 770 /home/sftp_root/shared
$ sudo vi /etc/ssh/sshd_config 
$ sudo reboot now
```

其中在sshd_config中最后面添加以下内容：

```
AllowGroups ssh-users sftp-users
Match Group sftp-users
    ChrootDirectory /home/sftp_root
    AllowTcpForwarding no
    X11Forwarding no
    ForceCommand internal-sftp
```

生成ssh key，在devstack的Vagranfile中使用到：  

```bash
$ mkdir ~/.ssh
$ ssh-keygen
```

安装NFS服务：  

```bash
$ sudo apt-get install nfs-common nfs-kernel-server
```

导入provision好的devstack box:   

```bash
$ vagrant box add dogwood-devstack-provisioned ~/Download/devstack_mac.box
```

## 启动虚拟机 ##

启动虚拟机：  

```bash
$ mkdir ~/openedx
$ cd openedx
$ git clone https://github.com/jinpeng/openedx/devstack
$ cd devstack
$ vagrant up
```

如果遇到ssh authentication问题：  

```bash
$ ssh-copy-id vagrant@192.168.33.10
$ vagrant reload
```

## 解决googleapis的问题 ##

由于众所周知的原因，open edX用到的fonts.googleapis.com，ajax.googleapis.com等

```bash
$ cd ~/openedx/devstack
$ grep -rl "ajax.googleapis.com" edx-platform | xargs sudo sed -i 's/ajax.googleapis.com/ajax.useso.com/g'
$ vagrant ssh
$ sudo vi /edx/app/edxapp/venvs/edxapp/lib/python2.7/site-packages/debug_toolbar/settings.py
```

## 启动Studio##

```bash
$ cd ~/openedx/devstack
$ vagrant ssh
$ sudo su edxapp
$ paver devstack studio
```

在host上打开浏览器，访问：  
http://192.168.33.10:8001/  
可以看到studio的登录界面，用预置账户staff@example.com / edx登录。


