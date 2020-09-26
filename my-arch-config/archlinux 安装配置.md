# archlinux 安装配置

<!--系统环境mbr+bios-->

## 1. 安装基本系统

### 1.1 WiFi联网

```shelll
iwctl
station wlan0 scan
station wlan0 connect wifiname
ping www.baidu.com
```

### 1.2 格式化分区并挂载

```
fdisk -l #查看分区列表
fdisk /dev/sda #格式化分区 ，点击选项m，查看help
#分区方案
mkfs.ext2 /dev/sda1 #/boot 200M
mkswap /dev/sda2 #swap 8G
mkfs.ext4 /dev/sda3 #/ 剩下所有空间
#挂载分区
mount /dev/sda3 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
swapon /dev/sda2
```

### 1.3 安装基本包

```
vim /etc/pacman.d/mirrorlist #修改国内源 /China 查找中国源，2dd剪切，gg到文件开头，p粘贴
pacman -Syy #更新源
pacstrap /mnt base base-devel linux firmware
```

### 1.4 生成分区表 并切换root

```
genfstab -U /mnt  >> /mnt/etc/fstab
arch-chroot /mnt
```

### 1.5 安装grub

```
pacman -S grub os-prober ntfs-3g
gurb-install /dev/sda
gurb-mkconfig -o /boot/gurb/grub.cfg
```

### 1.6 安装其他必备包

```
pacman -S iw wpa_supplicant dialog #WiFi联网必备
```

### 1.7 重启

```
exit #退出root
umount /mnt
reboot
```

### 1.8 时区  、语言

```
timedatectl set-ntp true
pacman -S vim
vim /etc/locale.gen #去掉en_US.UTF-8和zh_CN.UTF-8的注释
locale-gen #使生效
echo LANG=en_US.UTF-8 > /etc/locale.conf
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc --utc
```

### 1.9 驱动

```
pacman -S intel-ucode #intel 编码
pacman -S xf86-video-intel
```

### 1.10 组、用户

```
echo obsession > /etc/hostname #主机名obsession
vim /etc/hosts
	127.0.0.1 localhost
	::1 localhost
	127.0.1.1 obsession.localdomain obsession
useradd -m -g users -G wheel -s /bin/bash grace
pacman -S sudo 
visudo # 去掉%wheel ALL=(ALL)ALL 的注释，为wheel组添加sudo权限
passwd #root 密码
passwd grace #grace密码
```

### 1.11 装 X

```
pacman -S xorg-server xorg-xinit xfce4-trerminal
vim /etc/X11/xinit/xinitrc #删除twm（包含）之后的行
```

此时输入startx 应该有图形界面的terminal。

### 1.12  fcitx 输入法

```
pacman -S fcitx fcitx-im fcitx-configtool fcitx-configtool
yaourt fcitx-sogoupinyin
vim ~/.bashrc 和~/.xprofile         #写入配置文件，最好在root下写
	export GTK_IM_MODULE=fcitx
	export QT_IM_MODULE=fcitx
	export XMODIFIERS=“@im=fcitx”
```

### 1.13 多线程下载工具

#### 1.13.1 aria2c

```
sudo pacman -S aria2c
#修改pacman配置文件/etc/pacman.conf
#找到Xfercommand修改成如下:
XferCommand  = /usr/bin/aria2c -x 8 -s 8 --dir $(dirname %o) -o $(basename %o) %u

#保存更新系统
yaourt -Syyua
```



## 2 . 安装图形界面

### 2.1 yaourt 和yay

```
#安装yaourt
vim /etc/pacman.conf #最后加入
	[archlinuxcn]
	#The Chinese Arch Linux communities packages.
	SigLevel = Optional TrustAll
	Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch

pacman -S archlinuxcn-keyring
pacman -Syu yaourt
#配置AUR：
vim /etc/yaourttrc	#去掉AURURL注释，改为：
	AURURL="https://aur.tuna.tsinghua.edu.cn"

pacman -S yay

#系统全面更新
sudo pacman -Syyu --noconfirm
#登录后开启数字锁
yaourt -S --noconfirm systemd-numlockontty&&sudo systemctl enable numLockOnTty.service
```

之后yay 或者yaourt 安装即可。出现编辑时选n，安装选y。或者加入--noconfirm选项；若单个keyring未授权，则gpg --recv-keys  5DECDBA89270E723（签名）；若大范围签名出错，则重新初始化签名，并安装

```
  pacman -Syu haveged
  systemctl start haveged
  systemctl enable haveged
  rm -rf /etc/pacman.d/gnupg
  pacman-key --init
  pacman-key --populate archlinux
  pacman-key --populate archlinuxcn
```

### 2.2 安装常用软件

<!--本应在桌面环境建好之后写，但由于其通用性，故提前-->

```
yaourt -Sy --noconfirm netease-cloud-music smplayer smplayer-skins smplayer-themes google-chrome sublime-text-dev-zh-cn masterpdfeditor remarkable uget filezilla shadowsocks-qt5 deepin-screenshot shutter
gimp #类似PS
obs-studio #编辑视频音频
```

```
生产力
yaourt -Sy --noconfirm wiznote meld goldendict easystroke catfish peek gitkraken
			wiznote 为知笔记；
			meld 文本比较；
			goldendict 词典软件；
			easystroke 鼠标手势；
			GitKraken github 客户端
			catfish 基于GTK+的非常快速，轻量级的文件搜索工具；
			peek 屏幕录像工具，小巧玲珑，可保存录像为gif动图和兼容于html5的webm视频；
yaourt -Sy --noconfirm xmind  #有可能报错，选其他包即可
```



```
 yaourt -Sy --noconfirm bleachbit redshift 
     bleachbit 快速释放磁盘空间并不知疲倦地守卫你的隐私。释放缓存，删除 cookie，清除互联网浏览历史，清理临时文件，删除日志，以及更多功能...
    i-nex 小而全的系统信息查看软件；
    redshift 根据你的周边调整你屏幕的色温。当你夜晚在屏幕前工作时，它也许能帮助你减少对眼睛的伤害；
```

```
yaourt -Sy --noconfirm keepassx-git screenfetch-git freefilesync #需要网络git
    keepassx-git 密码管理器；
    screenfetch-git  系统信息工具，终端使用screenfetch命令；
    freefilesync 文件夹比较和同步工具；
```

```
yaourt -Sy --noconfirm eclipse-jee jetbrains-toolbox openjdk8-doc openjdk8-src dbeaver dbeaver-plugin-apache-poi dbeaver-plugin-batik dbeaver-plugin-office 

​			eclipse-jee 企业Java 集成开发环境；
​			jetbrains-toolbox 著名的jetbrains序列的IDE管理工具；
​			openjdk8-doc openjdk8-src 针对OpenJDK8的文档和源码；
​			dbeaver 通用数据库客户端，支持多个平台及多种数据库，社区版是免费的
```

```
yaourt -Sy --noconfirm codeblocks qtcreator glade glade-gtk2 kompozer kompozer-i18n-zh-cn			
	 	 codeblocks 跨平台的C++ IDE，官方网站上称其能满足最苛刻的用户的需求。
     qtcreator 基于QT的C++开发工具（包括界面设计）；
     glade 基于GTK3 的C++开发工具（包括界面设计）；
     glade-gtk2 基于GTK2 的C++开发工具（包括界面设计）；
     kompozer 类似Dreamweaver所见即所得功能的开源HTML编辑器。
```

```
yaourt -Sy --noconfirm nginx tomcat8 zookeeper
		 dbeaver 通用数据库客户端，支持多个平台及多种数据库，社区版是免费的；
    nginx 终端执行 sudo nginx 启动，sudo nginx -s stop/realod 停止或重启；
    tomcat8 开发必备，轻量的应用服务器；
    zookeeper 终端执行sudo zkServer.sh start 启动；
```

```
 yaourt -Sy --noconfirm cmatrix geogebra stellarium celestia
    cmatrix 终端从上往下输出无尽的字符串,类似<<黑客帝国>>中的矩阵效果，终端运行 cmatrix ；
    geogebra 图形计算器，支持函数，几何，代数，微积分，统计以及 3D 数学。
    stellarium 星象软件。可调选项很多，这是随便开起来截图的。
    celestia 免费的空间模拟器，让你在三维空间中探索我们的宇宙;
    gnucash 开源免费的个人或小型企业财务软件；
    gramps 家谱软件；
```

```
 yaourt -Sy --noconfirm nethack gnome-mines 2048-qt zaz

​    nethack 经典的命令行游戏，启动命令行nethack；
​    gnome-mines 经典的扫雷游戏（gnome桌面自带，kde也有类似的kmines）；
​    2048-qt 经典的2048游戏；
​    zaz 经典的泡泡射击游戏；

yaourt -Sy --noconfirm sudokuki wesnoth wesnoth-data 0ad

​    sudokuki 基于Java的跨平台的数独游戏					（https://sourceforge.net/projects/sudokuki/files/sudokuki/）；
​    wesnoth 经典的韦诺之战，Linux上比较火的游戏，回合制策略游戏；
​    0ad 跨平台的“帝国时代”（http://sourceforge.net/projects/zero-ad/files/releases/locales/下载对应版本的汉化放到$HOME/.local/share/0ad/mods/public/）；	
```

```
#虚拟机（全面更新系统重启后最后安装）	
sudo pacman -Sy virtualbox linux414-virtualbox-host-modules virtualbox-ext-oracle
    virtualbox 虚拟机工具，linux首选，比vmware还好用。
    https://wiki.manjaro.org/index.php?title=Virtualboxlinux414-virtualbox-host-modules  根据安装的内核版本选择，比如有 uname -r 如果是4.14内核，则安装 linux414-virtualbox-host-modules ；
```

### 2.3 Windows 可移植软件

```
vim /etc/pacman.conf #取消multilib注释，安装32位包，否则报错
yaourt -Sy deepin.com.thunderspeed #迅雷
yaourt -Sy deepin.com.qq.office #TIM
yaourt -Sy deepin.com.qq.im #qq
yaourt -Sy deepin.com.wechat  #wechat
yaourt -Sy deepin.com.baidu.pan

    #微信 QQ 输入法无法用，修改微信后QQ的启动脚本。
    #微信启动脚本：/opt/deepinwine/apps/Deepin-WeChat/run.sh
    #QQ启动脚本：/opt/deepinwine/apps/Deepin-TIM/run.sh
    #在脚本中添加如下配置：
        export GTK_IM_MODULE=fcitx
        export QT_IM_MODULE=fcitx
        export XMODIFIERS="@im=fcitx"

pacman -S wps-office ttf-wps-fonts  #wps
```

### 2.4 安装 gnome 桌面

#### 2.4.1安装gnome 基本包：

```
pacman -S gnome gnome-extra #其及依赖基本包含所有东西，声卡驱动，显卡驱动，触摸板驱动等等。
pacman -S ttf-dejavu wqy-microhei #安装字体，不然网页各处乱码
pacman -S networkmanager 
systemctl enable NetworkManager #重启生效
pacman -S gdm #登录管理器，不然得手动启动
systemctl enable gdm
```

重启，进入 gnome 图形界面

#### 2.4.2美化gnome

##### 2.4.2.1oh-my-zsh

```
sudo pacman -S zsh
sudo vim /etc/passwd #将要修改的用户的shell路径改为 /usr/bin/zsh 即可，也就是将 bash 改为 zsh
sudo pacman -S git wget curl #联网下载工具
#curl 方式：
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

#wget 方式
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"

#git方式：
git clone https://github.com/zsh-users/zsh-autosuggestions
cd ~/.oh-my-zsh/plugins
./install.sh

sudo chsh -s /bin/zsh grace
sudo vim ~/.zshrc 修改主题，找到 ZSH_THEME="robbyrussell"修改为 ZSH_THEME="random" 为随机主题，要换其他主题，修改此处即可
upgrade_oh_my_zsh #更新zsh
```

##### 2.4.2.2 gdm背景

```
curl -L -O http://archibold.io/sh/archibold
chmod +x archibold
./archibold login-backgroung 你的背景的地址
```

##### 2.4.2.3 主题，dash-to-dock

```
yaourt -S numix-circle-icon-theme-git
yaourt -S gtk-theme-arc-git
sudo pacman -S gnome-tweak-tool
#在tweak-tool里面启用主题及dash-to-dock
```





###  
