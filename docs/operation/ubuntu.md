## 安装ssh

**1：安装**

```bash
sudo apt update
sudo apt install openssh-server
```

**2：查看状态**

```bash
sudo systemctl status ssh
```

**3：连接服务器**

```bash
ssh username@ip_address
```

**4：配置免密登录**

```bash
blockchain@Dao:~$ ssh localhost
The authenticity of host 'localhost (127.0.0.1)' can't be established.
ECDSA key fingerprint is 70:a2:a9:3c:d3:b7:db:e3:eb:5a:e4:98:60:89:a2:ba.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.
blockchain@localhost's password: 
Permission denied, please try again.
blockchain@localhost's password: 
Permission denied, please try again.
blockchain@localhost's password: 
Permission denied (publickey,password).
blockchain@Dao:~$ 
123456789101112
```

此时，会要求输入密码，不用管，直接回车即可，进入刚生成的 .ssh 目录。

```bash
blockchain@Dao:~$ cd .ssh ; ls -lt
total 4
-rw-r--r-- 1 blockchain blockchain 222  8月 22 11:26 known_hosts
blockchain@Dao:~/.ssh$ 
1234
```

生成密钥，不用管提示，一直按回车。

```bash
blockchain@Dao:~/.ssh$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/blockchain/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/blockchain/.ssh/id_rsa.
Your public key has been saved in /home/blockchain/.ssh/id_rsa.pub.
The key fingerprint is:
50:7b:6c:91:31:bd:9c:c3:07:50:ef:b2:53:2d:e5:d3 blockchain@Dao
The key's randomart image is:
+--[ RSA 2048]----+
|        . =*.    |
|       . o.oo.   |
|      . . +o +. .|
|       . o  *..+.|
|        S   .o+.E|
|             + ..|
|            o    |
|             .   |
|                 |
+-----------------+
blockchain@Dao:~/.ssh$ ls -lt
total 12
-rw------- 1 blockchain blockchain 1679  8月 22 11:32 id_rsa
-rw-r--r-- 1 blockchain blockchain  396  8月 22 11:32 id_rsa.pub
-rw-r--r-- 1 blockchain blockchain  222  8月 22 11:26 known_hosts
1234567891011121314151617181920212223242526
```

加入授权

```bash
blockchain@Dao:~/.ssh$ cat ./id_rsa.pub >> ./authorized_keys
blockchain@Dao:~/.ssh$ ls -lt
total 16
-rw-rw-r-- 1 blockchain blockchain  396  8月 22 11:33 authorized_keys
-rw------- 1 blockchain blockchain 1679  8月 22 11:32 id_rsa
-rw-r--r-- 1 blockchain blockchain  396  8月 22 11:32 id_rsa.pub
-rw-r--r-- 1 blockchain blockchain  222  8月 22 11:26 known_hosts
1234567
```

测试免密码登录

```bash
blockchain@Dao:~$ ssh localhost
Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 4.4.0-133-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

0 packages can be updated.
0 updates are security updates.

New release '16.04.5 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Wed Aug 22 16:15:39 2018 from localhost
blockchain@Dao:~$ 
blockchain@Dao:~$ exit
logout
Connection to localhost closed.
blockchain@Dao:~$ 
1234567891011121314151617
```

免密码登录成功。

## 安装lrzsz

**1：安装lrzsz**

```bash
sudo apt install lrzsz
```

**2：sz命令发送文件到本地**

```bash
# sz filename
```

输入命令后会弹出接受文件选择目录

```bash
[root@jubeiTest data0]# sz nohup.out    
```

![img](https:////upload-images.jianshu.io/upload_images/7100414-a27d44b5ca72a614.png?imageMogr2/auto-orient/strip|imageView2/2/w/533/format/webp)



![img](https:////upload-images.jianshu.io/upload_images/7100414-10581f049a6c28c0.png?imageMogr2/auto-orient/strip|imageView2/2/w/360/format/webp)



**3：rz命令本地上传文件到服务器**

```bash
# rz
```

执行该命令后，在弹出框中选择要上传的文件即可。

![img](https:////upload-images.jianshu.io/upload_images/7100414-3c34c04f063aba9a.png?imageMogr2/auto-orient/strip|imageView2/2/w/568/format/webp)



上传到当前目录下，如何你要上传到/home/www下就直接先切换带目下

```bash
cd /home/www
rz 
```

![img](https:////upload-images.jianshu.io/upload_images/7100414-728bcfa47f8884a2.png?imageMogr2/auto-orient/strip|imageView2/2/w/349/format/webp)

备注：支持xshell和SecureCRT上传下载，其他没有试过，好像putty不支持！



## 配置静态ip

**方法一：在桌面中设置**



**方法二：命令设置**

Ubuntu20.04 配置值静态ip时需要修改/etc/netplan下面
1-network-manager-all.yaml这个文件，该文件的原始内容为：
![原始内容](https://img-blog.csdnimg.cn/20200610132429999.png)

在修改该文件时，先试用ifconfig查看下网卡相关信息，然后再对应修改，修改后的文件内容为：
![修改后的文件内容](https://img-blog.csdnimg.cn/20200610132600571.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NoYW9mYW5qdW4=,size_16,color_FFFFFF,t_70)
修改时相应的配置为：

```bash
ens33:   #配置的网卡名称
      dhcp4: no    #dhcp4关闭
      dhcp6: no    #dhcp6关闭
      addresses: [192.168.147.130/24]   #设置本机IP及掩码
      gateway4: 192.168.147.1   #设置网关
      nameservers:
          addresses: [192.168.147.1, 114.114.114.114]   #设置DNS
```

有时候在修改该文件时，会碰到无权限修改的情况，这时候使用chmod修改下文件权限即可。
修改完成后，在终端再输入

```bash
sudo netplan apply
```

此时，在终端中使用ifconfig查看ip时可以发现ip地址已经发生了变化。



## 更换软件源

```bash
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo vim /etc/apt/sources.list
```

Ubuntu 的软件源配置文件是 `/etc/apt/sources.list`。将系统自带的该文件做个备份，将该文件替换为下面内容，即可使用 TUNA 的软件源镜像。

20.04清华软件源：

```bash
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
```

清华源地址：https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/



## 安装英伟达驱动

### **1：安装 nvidia 驱动**

1. 开机按F2进入BIOS
2. security boot 设置disable
3. 参考https://blog.csdn.net/qq_34570910/article/details/78084659
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518210523468.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pfNl8yXzBfcw==,size_16,color_FFFFFF,t_70)

**方法一：安装NVIDIA驱动**

最开始安装驱动，首先禁止nouveau，然后卸载原先的nvidia驱动(如果有）

参考：https://blog.csdn.net/sinat_42239797/article/details/101618334?utm_medium=distribute.pc_relevant.none-task-blog-baidujs-1

但是装完出现这种情况：

- nvidia-smi有输出，nvidia-settings有反映，而且还生成了快捷图标，但是重启生效后，在设置->关于：显卡由原来的集成显卡630变成了lvib什么的，虽然不影响审定学习环境搭建但是总感觉以后会挂的。

- 还有一种情况是 ，装完成驱动后，在设置->关于：显卡显示GTX1060。但是每次开机或者关机显示：dev/sda5 clean …dev/sda6 clean.等2s后关机，开机也是这样。

- 还有一种情况是，环境搭建好了，驱动什么的都好了，但是一个命令，当时在安装网易云音月，要弄什么依赖，然后一行命令过去，开机无限闪现dev/sda6 clean 。ctro-alt-f1能打开tty,但是用户名和密码来不及输入，tty闪退，1s不到。然后进不了系统。最后重装系统

**解决方法：**
装完ubuntu系统后，什么更新都不要，也不要禁止nouveau。第一件事情直接装驱动，重启后，麻事情没有。

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518212210428.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pfNl8yXzBfcw==,size_16,color_FFFFFF,t_70)




**方法二：安装NVIDIA驱动**

转载原文：https://blog.csdn.net/xunan003/article/details/81665835
在这基础上补充了一些细节，并且在ubuntu20.04版本上实践可行

这种方式主要是为了避免sudo apt-get install nvidia-*安装方式造成**登录界面循环**。

1. ubuntu 默认安装了第三方开源的驱动程序nouveau，安装nvidia显卡驱动首先需要禁用nouveau，不然会碰到冲突的问题，导致无法安装nvidia显卡驱动。

编辑文件blacklist.conf：

```bash
sudo gedit /etc/modprobe.d/blacklist.conf
```

在文件最后部分插入以下两行内容

```ba'sh
blacklist nouveau
options nouveau modeset=0
```

更新系统

```bash
sudo update-initramfs -u
```

**重启系统（一定要重启）**

验证nouveau是否已禁用

```bash
lsmod | grep nouveau
```

没有信息显示，说明nouveau已被禁用，接下来可以安装nvidia的显卡驱动。

2. 在英伟达的官网上查找你自己电脑的显卡型号然后下载相应的驱动。网址：**http://www.nvidia.cn/page/home.html**

我下载的版本：NVIDIA-Linux-x86_64-450.57.run（注意不同的版本最后安装执行的具体选项不同）

下载后的run文件拷贝至home目录下。

3. 在ubuntu下按ctrl+alt+f1进入命令行界面，
   如果出现**unit lightdm.service not loaded**,则先更新apt-get

```bash
sudo apt-get update
```

然后安装lightdm

```bash
sudo apt-get install lightdm
```

安装后会跳出一个界面，选择**lightdm**
然后重启：**reboot**

重启登录后按ctrl+alt+F1进入命令行界面
**输入用户名和密码登录**后输入：

```bash
 sudo service lightdm stop      //这个是关闭图形界面，不执行会出错。
```

然后卸载掉原有驱动（如果装过的话）：

```bash
 sudo apt-get remove nvidia-*  （若安装过其他版本或其他方式安装过驱动执行此项）
```

4. 给驱动run文件赋予执行权限：

```bash
 sudo chmod  a+x NVIDIA-Linux-x86_64-396.18.run
```

安装：

```bash
 sudo ./NVIDIA-Linux-x86_64-396.18.run -no-x-check -no-nouveau-check -no-opengl-files 
 //只有禁用opengl这样安装才不会出现循环登陆的问题
```

-no-x-check：安装驱动时关闭X服务

-no-nouveau-check：安装驱动时禁用nouveau

-no-opengl-files：只安装驱动文件，不安装OpenGL文件

我讲一下我遇到的问题，我没遇到的问题可以参考我参照的原文
原文链接：https://blog.csdn.net/xunan003/article/details/81665835

(1)没有安装gcc、make
直接安装即可
sudo apt-get install gcc
sudo apt-get install make

(2)The distribution-provided pre-install script failed! Are you sure you want to continue?
选择 yes 继续。

Would you like to register the kernel module souces with DKMS? This will allow DKMS to automatically build a new module, if you install a different kernel later?
选择 No 继续。

问题没记住，选项是：install without signing
问题大概是：Nvidia’s 32-bit compatibility libraries?
选择 No 继续。

Would you like to run the nvidia-xconfigutility to automatically update your x configuration so that the NVIDIA x driver will be used when you restart x? Any pre-existing x confile will be backed up.
选择 Yes 继续

5. 安装完毕之后
   挂载Nvidia驱动：

```bash
modprobe nvidia
```

检查驱动是否安装成功：

```
nvidia-smi
```

如果出现如下提示，则说明安装成功：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200723105650152.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwNDY4NzIz,size_16,color_FFFFFF,t_70)

开启图形界面，重启

```bash
sudo service lightdm start
sudo reboot
```

安装完毕



### **2：解决安装驱动后循环登录**

**法一：关闭自动登录**

如果非要用 GPU，那只能放弃 ubuntu 系统的自动登录功能，参照这个解决：
原文：[解决ubuntu 20.04安装nvidia驱动后循环登录的问题](https://www.cnblogs.com/tengge/p/12774341.html)

1. 登录界面，按Ctrl + Alt + F1 或 F2 或 F3，进入终端，登录
2. 查看默认显示管理器

```bash
cat /etc/X11/default-display-manager
```

如果是 LightDM，则显示

```bash
/usr/sbin/lightdm
```

如果是 GDM3，则显示

```bash
/usr/sbin/gdm3
```

如果是 SDDM，则显示

```bash
/usr/sbin/sddm
```

3. 编辑文件

如果是 LightDM，输入

```bash
sudo vi /etc/lightdm/lightdm.conf
```

注释自动登录代码（自己去搜 vi 的操作方法）

```bash
#autologin-user=<user_name>
#autologin-user-timeout=0
```

如果是 GDM3，输入

```bash
sudo vi /etc/gdm3/custom.conf
```

注释自动登录代码（自己去搜 vi 的操作方法）

```bash
# AutomaticLoginEnable = true
# AutomaticLogin = <user_name>
```

如果是 SDDM，输入

```bash
sudo vi /etc/sddm.conf.d/autologin.conf
```

注释自动登录代码（自己去搜 vi 的操作方法）

```bash
#[Autologin]
#User=<user_name>
```

4. 重启即可

```bash
reboot
```

**法二：卸载显卡驱动**

如果比较介意没有自动登录功能，但又不介意没有 GPU，参照这个解决：
原文：[Ubuntu桌面等不进去？](https://www.zhihu.com/collection/528416280)

1. 登录界面，按Ctrl + Alt + F1 或 F2 或 F3，进入终端，登录
2. 清理 nvidia 驱动残留

```bash
sudo apt-get purge nvidia*
```

3. 清理系统

```bash
sudo apt-get autoremove
```

4. 重装 ubuntu-desktop

```bash
sudo apt-get install --reinstall ubuntu-desktop
```

5. 重启即可

```bash
sudo reboot
```





## Ubuntu 20.04 桌面美化



### 1：修改字体和调整缩放

1.1 安装 gnome-tweak-tools

```bash
sudo apt install gnome-tweak-tools
```

1.2 启动 gnome-tweak-tools

在菜单中选择`优化`条目启动

### 2：安装 GNOME Shell 扩展

火狐浏览器安装 GNOME Shell 扩展如下步骤：

1. 安装chrome-gnome-shell

```bash
sudo apt install chrome-gnome-shell
```

2. 在火狐浏览器中安装插件GNOME Shell integration

![img](https://img-blog.csdnimg.cn/20200501194827303.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2Mjg1OTk3,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20200501195635379.png)

3. 点击浏览器右上角插件图标或 直接打开 [https://extensions.gnome.org/ ](https://extensions.gnome.org/)安装gnome扩展

![img](https://img-blog.csdnimg.cn/20200501195707930.png)

4. 在https://extensions.gnome.org/页面中，选择或搜索要安装的gnome扩展（主题也可以哦）

![img](https://img-blog.csdnimg.cn/20200501200105647.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2Mjg1OTk3,size_16,color_FFFFFF,t_70)

以 user themes 为例，点击进入，如要安装，点击右侧on/off 开关，即可自行安装。

![img](https://img-blog.csdnimg.cn/20200501200038266.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2Mjg1OTk3,size_16,color_FFFFFF,t_70)

5. 最后上个经过扩展后的最终界面

![img](https://img-blog.csdnimg.cn/20200501201547248.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2Mjg1OTk3,size_16,color_FFFFFF,t_70)

这里没有安装任何主题，只添加了`dash to panel`扩展（把顶栏与底栏合并）。

### 3：安装 dash to panel

在 [火狐扩展](https://extensions.gnome.org/) 中 选择`dash to panel` ，然后点击 `on` 按钮。



### 4：安装 deepin-wine

安装必要的工具及deepin-wine依赖



```
sudo apt install wget g++ git     #如已安装可自行跳过
```

**2.安装deepin-wine**



```
git clone "https://gitee.com/wszqkzqk/deepin-wine-for-ubuntu.git"
cd deepin-wine                    #切换到下载目录
sudo ./install.sh                 #执行安装
```

deepin-wine容器安装完成，下面进行具体软件的安装。

**3.安装Deep-wine微信及QQ**

微信



```
sudo wget "https://mirrors.huaweicloud.com/deepin/pool/non-free/d/deepin.com.wechat/deepin.com.wechat_2.6.2.31deepin0_i386.deb" 
sudo dpkg -i *wechat*deb              #安装微信
sudo apt install libjpeg62:i386       #解决微信无法查看发送图片问题
```

QQ



```
sudo wget https://mirrors.aliyun.com/deepin/pool/non-free/d/deepin.com.qq.im/deepin.com.qq.im_9.1.8deepin0_i386.deb
sudo dpkg -i *qq.im*deb
```

TIM



```
sudo wget https://mirrors.aliyun.com/deepin/pool/non-free/d/deepin.com.qq.office/deepin.com.qq.office_2.0.0deepin4_i386.deb
sudo dpkg -i *qq.office*deb
```

如有其它软件需求可使用deepin发布的最新版容器安装包：

1. [QQ轻聊版](https://link.zhihu.com/?target=https%3A//mirrors.aliyun.com/deepin/pool/non-free/d/deepin.com.qq.im.light/)
2. [Foxmail](https://link.zhihu.com/?target=https%3A//mirrors.aliyun.com/deepin/pool/non-free/d/deepin.com.foxmail/)
3. [百度网盘](https://link.zhihu.com/?target=https%3A//mirrors.aliyun.com/deepin/pool/non-free/d/deepin.com.baidu.pan/)
4. [WinRAR](https://link.zhihu.com/?target=https%3A//mirrors.aliyun.com/deepin/pool/non-free/d/deepin.cn.com.winrar/)
5. [迅雷极速版](https://link.zhihu.com/?target=https%3A//mirrors.aliyun.com/deepin/pool/non-free/d/deepin.com.thunderspeed/)

其它deepin-wine容器：[阿里云镜像下载](https://link.zhihu.com/?target=https%3A//mirrors.aliyun.com/deepin/pool/non-free/d/)

4.托盘悬浮

正常安装后wine窗口为独立窗口，为了便于美观建议安装如下插件：

Gnome Shell 插件：[TopIcons Plus](https://link.zhihu.com/?target=https%3A//extensions.gnome.org/extension/1031/topicons/)

[![img](https://pic2.zhimg.com/80/v2-c34a6a292d7d660d40098a6f4e987b55_720w.png)](https://pic2.zhimg.com/80/v2-c34a6a292d7d660d40098a6f4e987b55_720w.png)安装后效果图

**5.软件需求其他辅助软件的安装方法**

1. 下载需要的软件安装包，exe文件，如：flash
2. 将下载的安装文件放入 ~/.deepinwine/<容器名（微信的为Deepin-WeChat，TIM为：Deepin-TIM）>/drive_c 下，即软件所在 Wine C 盘根目录
3. 打开一个 Terminal ，执行：



```
WINEPREFIX=~/.deepinwine/<容器名> deepin-wine "c:\\<文件名>"
```

然后按提示进行安装、重启软件即可。

**6.手动更改配置（winecfg）**

执行以下代码，并根据需求进行配置更改。



```
WINEPREFIX=~/.deepinwine/容器名称 deepin-wine winecfg 
```

**7.卸载方法**



```
uninstall.sh
```

**8.系统非中文语言环境时软件设置为中文**

修改/opt/deepinwine/tools/run.sh 文件，将 WINE_CMD 那一行修改为 WINE_CMD="LC_ALL=zh_CN.UTF-8 deepin-wine"



```
sudo vim /opt/deepinwine/tools/run.sh       #打开文件进行修改
```

**9.软件更新**



```
wget -qO- https://deepin-wine.i-m.dev/setup.sh | sudo sh
sudo apt-get install deepin.com.qq.office      #安装/更新TIM
sudo apt-get install deepin.com.qq.im          #安装/更新QQ
sudo apt-get install deepin.com.wechat         #安装/更新微信
```

**10.wine全部进入后台后无法调用问题****1. 安装 xdotool**



```
sudo apt install --no-install-recommends xdotool
```

**2. 编写 xdotool 脚本**

*思路: Wine 应用在后台无法接收到快捷键状态, 此时借助 xdotool 向 Wine 应用发送模拟按键信息即可. *

在合适的位置新建一个脚本文件 "open_wechat.sh", 写入以下内容:



```
#!/bin/sh 
#在当前运行的应用中找到名为WeChat.exe的应用程序，并向它发送按键事件"ctrl+alt+W" 
#WeChat的可执行文件名为WeChat.exe，如果是其它应用程序就修改成其它应用程序的可执行文件名, 应用名称大小写敏感, 一个字母都不能错! 
xdotool key --window $(xdotool search --limit 1 --all --pid $(pgrep WeChat.exe)) "ctrl+alt+W"
```

赋予脚本可执行权限:



```
chmod +x open_wechat.sh
```

如果此时你的微信正好运行在后台, 执行这个脚本就可以把它召唤到前台. 如果没有, 请检查脚本是否有错误.

**3. 设置快捷键**

图形界面依次打开 "设置" -> "设备" -> "键盘快捷键", 点击列表最底部的 "+" 号添加自定义快捷键.

- 名称随便, 填写 "打开微信" 即可;
- 命令填写刚才编写的脚本的**全路径**;
- 快捷键设置自己想用的快捷键即可, 建议于应用内部快捷键相同;
- 最后点击"添加"即可.

**4. 验证**

到这里已经设置成功了, 打开微信, 切换到后台, 然后按下刚才设置的快捷键就能召唤应用至前台. 如果不能, 请检查自己前面的设置是否有误.

目前存在无法语音通话和视频



### 5：xshell中修改主机名颜色

**windows下解决**

1. 打开xshell当前会话属性或者默认会话属性（如下图）

   ![img](https:////upload-images.jianshu.io/upload_images/256855-c61bc80c77e4e791.png?imageMogr2/auto-orient/strip|imageView2/2/w/779/format/webp)

2. 修改外观中的突出显示，利用正则表达式选中username@hostname从而修改其颜色（见下方操作）。

   ![img](https:////upload-images.jianshu.io/upload_images/256855-a7a2a15de0a96f1c.png?imageMogr2/auto-orient/strip|imageView2/2/w/873/format/webp)

- 正则表达式：由于`username@hostname`在`[]`中，在我的电脑上是[hdp@hadoop-server mapreduce]，所以直接使用了`\[.*\]`的正则

**linux下解决**

1. 修改用户目录下的`.bashrc`文件，每个用户家目录下的对应文件都需要修改（修改一个只对应一个用户的，其他用户的不会变），`vim .bashrc`然后做如下修改



```shell
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi

# Uncomment the following line if you don't like systemctl's auto-paging feature:
# export SYSTEMD_PAGER=

# User specific aliases and functions
#上面是centos中原来的内容，下面是新加的（Ubuntu中有下面的内容，去掉一行注释就行，具体我也不知道啥道理，只是试了可以用）


# set a fancy prompt (non-color, unless we know we "want" color)
case "$TERM" in
    xterm-color|*-256color) color_prompt=yes;;
esac
 
# uncomment for a colored prompt, if the terminal has the capability; turned
# off by default to not distract the user: the focus in a terminal window
# should be on the output of commands, not on the prompt
force_color_prompt=yes   #Ubuntu中只需要将这行的注释去掉就行
 
if [ -n "$force_color_prompt" ]; then
    if [ -x /usr/bin/tput ] && tput setaf 1 >&/dev/null; then
    # We have color support; assume it's compliant with Ecma-48
    # (ISO/IEC-6429). (Lack of such support is extremely rare, and such
    # a case would tend to support setf rather than setaf.)
    color_prompt=yes
    else
    color_prompt=
    fi
fi
 
if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
fi
unset color_prompt force_color_prompt
```

2.执行`source .bashrc`使修改生效

**结果：**

![img](https:////upload-images.jianshu.io/upload_images/256855-bd13e11f6ab84b7d.png?imageMogr2/auto-orient/strip|imageView2/2/w/983/format/webp)

### 6：安装中文输入法

1. 之前18.04版本，安装的是fcitx，升级到20.04后，输入法失效。

2. 删除fcitx

```bash
sudo apt remove fcitx
```

3. 安装ibus-libpinyin

```bash
sudo apt install ibus-libpinyin 
sudo apt install ibus-clutter
```

4. 安装好后，可以在Chrome浏览器中输入中文了。（可能需要reboot，忘了）

5. 在应用程序中找到“语言支持”（可搜关键字 region或language），更改"键盘输入法系统"，  改为“IBUS“(之前是18.04系统，选的是fcitx)。

![键盘输入法系统改为“IBUS”](https://img-blog.csdnimg.cn/2020040711042549.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zfem9uZ2ppYW4=,size_16,color_FFFFFF,t_70)

6. 重启系统(不是注销)，然后Chromium就可以输入中文了。

