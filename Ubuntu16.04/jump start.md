- [Ubuntu16.04装机](#ubuntu1604--)
  * [制作启动盘](#-----)
  * [分区](#--)
  * [apt换源](#apt--)
  * [配置tmux、vim](#--tmux-vim)
  * [硬盘挂载](#----)
  * [安装teamviewer](#--teamviewer)
  * [安装ssh和远程桌面](#--ssh-----)
    + [安装ssh](#--ssh)
    + [安装远程桌面](#------)
    + [配置](#--)
      - [新机器设置端口转发：hiwifi.com 互联网-超级端口转发](#----------hiwificom-----------)
      - [不是新机器设置静态ip](#---------ip)
      - [固件错误Possible missing firmware解决:](#----possible-missing-firmware---)
        * [1、进入如下这个地址，固件文件非常全面，找到适合自己的版本](#1----------------------------)
        * [2、切换到刚才报缺少固件的目录，下载对应的文件内容，](#2-------------------------)
  * [安装驱动](#----)
    + [1、禁用nouveau第三方驱动](#1---nouveau-----)
    + [2、重启后按Ctrl+Alt+F1 进入命令行界面](#2-----ctrl-alt-f1--------)
  * [安装cuda10.1 （10.0兼容性不好）](#--cuda101--100------)
  * [安装Cudnn10.0](#--cudnn100)
  * [安装anaconda3（导入设置）](#--anaconda3------)
    + [配置](#---1)
  * [pip换源](#pip--)
  * [添加用户](#----)
  * [安装mujoco](#--mujoco)

# Ubuntu16.04装机

**2080Ti Cuda10.1 Cudnn7.6.5**

## 制作启动盘

镜像源：https://mirrors.tuna.tsinghua.edu.cn/ubuntu-releases/16.04/
下载：ubuntu-16.04.6-desktop-amd64.iso
使用UltraISO：打开iso文件—启动—写入硬盘映像

## 分区

下载时选择something else, 分区如下（500G+2T）
![]([https://github.com/ReedZyd/ReedPages/blob/master/Ubuntu16.04/ubuntu%E8%A3%85%E6%9C%BA%E5%88%86%E5%8C%BA.jpg](https://github.com/ReedZyd/ReedPages/blob/master/Ubuntu16.04/ubuntu装机分区.jpg)

## apt换源

```shell
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo gedit /etc/apt/sources.list
```

按系统版本替换内容：
https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/
更新升级

```shell
sudo apt update
sudo apt upgrade
```

## 配置tmux、vim

```shell
sudo apt-get install tmux
sudo apt-get install vim
sudo apt-get install git
sh <(curl https://j.mp/spf13-vim3 -L) # 推荐配置，容易连不上，多试几次
```

## 硬盘挂载

```shell
sudo fdisk -l #查看可挂载的磁盘都有哪些
df -h  #查看已经挂载了哪些磁盘
sudo mkdir /DATA 
sudo mkfs -t ext4 /dev/sda
sudo mount /dev/sda /DATA
vim /etc/fstab 
```

添加到最后一行：UUID=*************  /DATA  ext4  defaults  0  1 
(ls -l /dev/disk/by-uuid | grep sda查看UUID)

## 安装teamviewer

`sudo dpkg -i xxx.deb` 若缺少依赖：sudo apt install -f

## 安装ssh和远程桌面

### 安装ssh

```shell
sudo apt install openssh-server
sudo service ssh start
sudo service ssh status
```

### 安装远程桌面

```shell
sudo add-apt-repository ppa:hermlnx/xrdp #添加高版本的xrdp源
sudo apt-get update
sudo apt-get install xrdp  #安装xrdp 
sudo apt-get install vnc4server #安装vnc4server 
sudo apt-get install xubuntu-desktop #安装xubuntu-desktop
echo xfce4-session >~/.xsession
sudo service xrdp restart
sudo reboot
```

全灰屏，鼠标是个叉，可能是因为xrdp版本低，参考：

https://netdevops.me/2017/installing-xrdp-0.9.1-on-ubuntu-16.04-xenial/

没有共享剪切板也是因为版本低（官方Ubuntu16.04的源里只有0.6.1-2的版本）

没有菜单栏、tab补全等：https://www.cnblogs.com/defineconst/p/10254613.html

### 配置

#### 新机器设置端口转发：hiwifi.com 互联网-超级端口转发

#### 不是新机器设置静态ip

- 1、查询网络接口的名字
  打开命令行，输入`ifconfig`,第一行最左边的名字，就是本机的网络接口，如enp5s0 

- 2、打开修改文件
  `sudo vim /etc/network/interfaces`
  添加：

  > auto enp5s0 # 使用的网络接口，之前查询接口是为了这里  
  > iface enp5s0 inet static // enp5s0这个接口，使用静态ip设置  
  > address 192.168.199.105 // 设置ip地址  
  > netmask 255.255.255.0 // 设置子网掩码  
  > network 192.168.199.0
  > broadcast 192.168.199.255
  > gateway 192.168.199.1 // 设置网关  
  > dns-nameservers 114.114.114.114 // 设置dns服务器地址

- 3、刷新ip
  单纯使用断开连接再重新连接，并不是正确的方式，应该使用以下命令行。

  ```shell
  sudo ip addr flush enp5s0
  sudo systemctl restart networking.service
  ```

- 4、sudo reboot

- 5、修改设置

  ```shell
  sudo gedit /etc/NetworkManager/NetworkManager.conf
  ```

  ```
  将“managed=false”修改为“managed=true”
  
  ```

- 6、重启服务`sudo service network-manager restart`

- 7、配置dns-nameserver 

  - 暂时配置：
    `sudo vim /etc/resolv.conf`
    添加 `nameservers 114.114.114.114`

  - 永久配置：

    ```shell
    sudo –i  
    cd /etc/resolvconf/resolv.conf.d  
    vim base
    
    ```

    添加`nameserver 114.114.114.114`

#### 固件错误Possible missing firmware解决: 

##### 1、进入如下这个地址，固件文件非常全面，找到适合自己的版本

https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/tree/rtl_nic/

##### 2、切换到刚才报缺少固件的目录，下载对应的文件内容，

```shell
cd /lib/firmware/rtl_nic/
sudo wget https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/tree/rtl_nic/rtl8125a-3.fw

```

## 安装驱动

### 1、禁用nouveau第三方驱动

`sudo gedit /etc/modprobe.d/blacklist.conf`
在最后一行添加：blacklist nouveau

```shell
sudo update-initramfs -u
sudo reboot

```

### 2、重启后按Ctrl+Alt+F1 进入命令行界面

执行命令：lsmod | grep nouveau 查看是否禁用,无反应则已禁用
禁用界面：sudo /etc/init.d/lightdm stop 关闭桌面

```shell
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt-get update
ubuntu-drivers devices # 查询所有ubuntu推荐的驱动
sudo apt-get install nvidia-430
sudo /etc/init.d/lightdm start
watch -n 0.1 -d nvidia-smi #查看显卡温度 不跑程序时30-50C均可 刚装好驱动时会热一点 多等一会儿

```

## 安装cuda10.1 （10.0兼容性不好）

下载`.deb`文件：

```shell
sudo dpkg -i xxx
sudo apt-key add /var/cuda-repo-<version>/7fa2af80.pub #见上条运行结果
sudo apt-get update
sudo apt-get install cuda
sudo gedit ~/.bashrc

```

添加：

```
export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
export CUDA_HOME=/usr/local/cuda

```

```shell
source ~/.bashrc

```

## 安装Cudnn10.0

有时候需要改名： solitairetheme8-->tgz

```shell
tar -zxvf xxx.tgz 
sudo cp cuda/include/cudnn.h /usr/local/cuda/include/
sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64/ -d

```

查看CUDA版本`cat /usr/local/cuda/version.txt`
查看 CUDNN 版本：`cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2`

## 安装anaconda3（导入设置）

### 配置

```shell
echo 'export PATH="~/anaconda3/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

```

## pip换源

```shell
pip install pip -U
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

```

## 添加用户

- 添加用户：`sudo addusr xxx`
- 添加root权限：
  `sudo vim /etc/sudoers`
  添加：`xxx ALL=(ALL) ALL`

## 安装mujoco

- 安装： Follow https://github.com/openai/mujoco-py
- 往~/.bashrc添加环境变量：

```
export LD_LIBRARY_PATH=~/.mujoco/mujoco200/bin${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}

```

```shell
cp mjkey.txt ~/.mujoco 
cp mjkey.txt ~/.mujoco/mujoco200/bin

```

```
Export LD_LIBRARY_PATH=~/.mujoco/mujoco200/bin${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}

```

```
cd ~/.mujoco/mjpro150/bin 
./simulate ../model/humanoid.xml

```

- Install mujoco-py 参考 https://blog.csdn.net/zhangkzz/article/details/84574772
