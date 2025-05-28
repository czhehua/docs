# 主题：Debian / Ubuntu LTS 服务器环境、桌面开发环境配置

**Revision:** 1.0  
**最后更新:** 2025/05/13

- [主题：Debian / Ubuntu LTS 服务器环境、桌面开发环境配置](#主题debian--ubuntu-lts-服务器环境桌面开发环境配置)
  - [1. 需求](#1-需求)
  - [2. 服务器环境](#2-服务器环境)
    - [2.1. Debian 服务器的安装和配置](#21-debian-服务器的安装和配置)
    - [2.1.1. 语言和键盘设置](#211-语言和键盘设置)
    - [2.1.2. 磁盘分区](#212-磁盘分区)
    - [2.1.3. 设置 root 和 默认用户](#213-设置-root-和-默认用户)
    - [2.1.4. 安装额外的组件](#214-安装额外的组件)
    - [2.1.5. 安装完成后的设置](#215-安装完成后的设置)
      - [2.1.5.1. 将用户加入 sudo 用户组](#2151-将用户加入-sudo-用户组)
      - [2.1.5.2. 其他用户组相关操作](#2152-其他用户组相关操作)
      - [2.1.5.3. 设置 LightDM 登录界面显示用户名](#2153-设置-lightdm-登录界面显示用户名)
      - [2.1.5.4. 文件管理器 Thunar 增加 SMB 访问支持](#2154-文件管理器-thunar-增加-smb-访问支持)
    - [2.2. Ubuntu LTS 服务器的安装和配置](#22-ubuntu-lts-服务器的安装和配置)
    - [2.2.1. 语言和键盘设置](#221-语言和键盘设置)
    - [2.2.2. 网络配置](#222-网络配置)
    - [2.2.3. 磁盘分区设置](#223-磁盘分区设置)
    - [2.2.4. 安装 SSH Server](#224-安装-ssh-server)
    - [2.2.5. 安装完成后的设置](#225-安装完成后的设置)
      - [2.2.5.1. 设置 root 密码](#2251-设置-root-密码)
      - [2.2.5.2. 安装桌面环境](#2252-安装桌面环境)
      - [2.2.5.3. 卸载 Snap](#2253-卸载-snap)
      - [2.2.5.4. 安装浏览器、中日韩字体](#2254-安装浏览器中日韩字体)
  - [3. 桌面开发环境](#3-桌面开发环境)
    - [3.1. Ubuntu 桌面版的安装](#31-ubuntu-桌面版的安装)
    - [3.2. Ubuntu 桌面版安装后的设置](#32-ubuntu-桌面版安装后的设置)
      - [3.2.1. 修改 user-dirs 目录名为英文](#321-修改-user-dirs-目录名为英文)
      - [3.2.2. 安装浏览器](#322-安装浏览器)
      - [3.2.3. 安装小企鹅输入法 fcitx5](#323-安装小企鹅输入法-fcitx5)
      - [3.2.4 完全卸载并清理 Snap 服务](#324-完全卸载并清理-snap-服务)

## 1. 需求 

Debian / Ubuntu LTS 版本自发布日起，一般有5年维护期，较适用于作为长期稳定的服务器环境，或桌面开发环境。  
本文以 Debian 12 / Ubuntu 24.04 为例，记录一些安装及配置的要点。

## 2. 服务器环境 

### 2.1. Debian 服务器的安装和配置

**注意：** 
- **Debian 版本和支持期限：**
  打开 [Debian Release Wiki](https://wiki.debian.org/DebianReleases) 页面，查看最新的 Debian 版本，并确认 `End of life date` 日期。
- 使用 DVD 镜像（可离线）：[ISO-DVD](https://cdimage.debian.org/debian-cd/current/amd64/iso-dvd/)
- 使用 网络安装镜像（需联网）：[ISO-CD](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/)

Debian 不区分服务器或桌面版，使用上述下载的ISO镜像安装即可。

安装时，可参考 [官方安装手册](https://www.debian.org/releases/stable/amd64/index.zh-cn.html)，下面列出一些注意点：

### 2.1.1. 语言和键盘设置

Debian 服务器安装建议如下设置：

- Language：默认 English
- Location：选择 other - Asia - China
- Locale： 默认 en_US.UTF-8
- Keymap： 默认 American English

### 2.1.2. 磁盘分区

磁盘分区：可整盘安装，或按如下创建新分区：  
- 使用 BIOS 引导：创建 swap 和根分区：/, swap  
- 使用 UEFI 引导：创建 efi, swap 和根分区：/boot/efi, /, swap  

### 2.1.3. 设置 root 和 默认用户

安装过程中，建议设置 root 用户的密码、默认用户的用户名和密码。

### 2.1.4. 安装额外的组件

运行到 `tasksel` 阶段，安装轻量级的 Xfce 桌面。 (可选)
勾选 SSH Server 以开启远程访问。  
- [x] Debian desktop environment  
- [x] Xfce  
- [x] SSH server  

### 2.1.5. 安装完成后的设置

#### 2.1.5.1. 将用户加入 sudo 用户组

运行如下命令，将当前用户加入到 sudo 用户组。然后注销后重新登录即可：

```bash
su -
# 输入 root 用户的密码（在安装时已设置）
usermod -aG sudo $USER
```

#### 2.1.5.2. 其他用户组相关操作

安装 Docker 后，可将当前用户加入到 docker 用户组，方便运行：

```bash
sudo usermod -aG docker $USER
```

如后续服务器注册了 gitlab-runner，并调用 docker，可将其也加入到 docker 用户组：

```bash
sudo usermod -aG docker gitlab-runner
```

#### 2.1.5.3. 设置 LightDM 登录界面显示用户名

**注意：** 仅当使用 Xfce 等桌面时需要配置 LightDM。如使用 gdm 或 sddm，则无需配置。

编辑 /etc/lightdm/lightdm.conf，可设置 lightdm 登录界面上显示用户名：

```
[Seat:*]
...
greeter-hide-users=false
```

其他 LightDM 相关配置，可参照 [Debian Wiki LightDM]([LightDM](https://wiki.debian.org/LightDM)) 页面。


#### 2.1.5.4. 文件管理器 Thunar 增加 SMB 访问支持

某些情况下，从 文件管理器 Thunar 中访问 `smb://` 开头的网络共享文件夹无效，需要安装 `gvfs-backends` 才能访问。

```bash
sudo apt install gvfs-backends
```

### 2.2. Ubuntu LTS 服务器的安装和配置

**注意：** 
- **Ubuntu 服务器应安装 LTS 版本：**
  打开 [Ubuntu Release Wiki](https://wiki.ubuntu.com/Releases) 页面，查看最新的 Ubuntu LTS 版本，并确认 `End of Standard Support` 日期。
- 下载 [Ubuntu Server 24.04.x (Noble Numbat) Daily Build](https://cdimage.ubuntu.com/ubuntu-server/noble/daily-live/current/noble-live-server-amd64.iso)
- 或访问 [Ubuntu Server 下载页](https://ubuntu.com/download/server) 获取最新 LTS 版本。

安装时，可查看 [官方安装教程](https://ubuntu.com/tutorials/install-ubuntu-server#1-overview)，下面列出一些注意点：

### 2.2.1. 语言和键盘设置

Ubuntu 服务器安装建议如下设置：

- Language：默认 English
- Keyboard： 默认 English (US)

### 2.2.2. 网络配置

Ubuntu 服务器的网络配置可按如下配置（配置有线网卡 `ensXXX` ）：

- **IPv4 Method:** Manual
- **Subnet:** 172.16.0.0/16
- **Address:** 172.16.103.253 （※根据实际申请到的服务器IP地址设置）
- **Gateway:** 172.16.0.254
- **Name servers:** 172.16.0.254

### 2.2.3. 磁盘分区设置

一般保持默认 `Use an entire disk` 即可。可取消 LVM 相关选项。  
如安装多系统，则选择 `Custom storage layout` 进行自定义设置。

### 2.2.4. 安装 SSH Server

勾选安装 SSH Server 以开启远程访问。   

**注意：** 
- 联网状态，Featured Server snaps 页面，可选安装 Docker 等服务，不要选择安装，因为 Snap 版本运行在沙盒环境中，后续使用中会有限制，维护起来不方便。

### 2.2.5. 安装完成后的设置

#### 2.2.5.1. 设置 root 密码

可设置 root 密码，方便后续管理。

```bash
sudo passwd root
```

#### 2.2.5.2. 安装桌面环境

服务器环境可选择安装轻量级的 lxqt-core 桌面，方便后续维护。 (可选)

```bash
sudo apt update
sudo apt install lxqt-core sddm
```

#### 2.2.5.3. 卸载 Snap

如不使用 Snap 服务可卸载，以防后续误安装 Snap 版服务。

参考 [完全卸载 Snap 服务](#324-完全卸载-snap-服务)

#### 2.2.5.4. 安装浏览器、中日韩字体


服务器环境安装浏览器，可参照 [Ubuntu 安装 deb 版 Firefox](Ubuntu替换Snap版Firefox.md#3-使用-deb-包安装-firefox)  

或安装 Google Chrome 浏览器。 (可选)

```bash
cd ~/Downloads
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo apt install ./google-chrome-stable_current_amd64.deb
```

如需浏览中文或日文网页，可安装中日韩字体。 (可选)

```bash
sudo apt install fonts-noto-cjk
```

## 3. 桌面开发环境 

桌面开发环境以安装 Ubuntu 24.04 为例说明。

### 3.1. Ubuntu 桌面版的安装

**注意：** 
- **Ubuntu 桌面开发环境也推荐安装 LTS 版本：**
  由于非 LTS 版本只有9个月维护期，因此作为稳定的开发环境，也推荐使用 LTS 版本。
  打开 [Ubuntu Release Wiki](https://wiki.ubuntu.com/Releases) 页面，查看最新的 Ubuntu LTS 版本，并确认 `End of Standard Support` 日期。
- 下载 [Ubuntu 24.04.x (Noble Numbat) Daily Build](https://cdimage.ubuntu.com/noble/daily-live/current/noble-desktop-amd64.iso)
- 或访问 [Ubuntu 下载页](https://ubuntu.com/download/desktop) 获取最新 LTS 版本。

从 ISO 引导后，会自动进入安装程序，按向导安装即可。

### 3.2. Ubuntu 桌面版安装后的设置

#### 3.2.1. 修改 user-dirs 目录名为英文

Ubuntu 安装时，如果语言选择的是中文，安装后用户主目录下的 user-dirs 默认也是中文目录名，可按如下方法切换回英文。 (可选)

```bash
sudo apt install xdg-user-dirs-gtk

# 临时切换系统语言为英文
export LANG=en_US

xdg-user-dirs-gtk-update
# 会有个窗口提示语言更改，更新成英文新名称

# 将系统语言切换回中文 (可选)
export LANG=zh_CN.UTF-8

reboot
```

重启电脑后再次登录，若提示语言更改，选择保留旧的英文名文件夹即可。

#### 3.2.2. 安装浏览器

桌面环境推荐替换 Firefox 浏览器，可参照 [Ubuntu 安装 deb 版 Firefox](Ubuntu替换Snap版Firefox.md#3-使用-deb-包安装-firefox)  

如有需要，也可选择安装 Google Chrome 浏览器。 (可选)

```bash
cd ~/Downloads
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo apt install ./google-chrome-stable_current_amd64.deb
```

#### 3.2.3. 安装小企鹅输入法 fcitx5

输入中文或日文，推荐使用小企鹅输入法 fcitx5 输入法。可输入下面的命令安装。

```bash
# 卸载 ibus 和 fcitx4 输入法，防止冲突
sudo apt purge ibus-* fcitx-*

# 安装 fcitx5 以及中日文输入法
sudo apt install fcitx5 fcitx5-frontend-all fonts-noto-cjk
sudo apt install fcitx5-chinese-addons fcitx5-module-cloudpinyin 
sudo apt install fcitx5-mozc mozc-utils-gui 
```

设置环境变量，编辑或创建 .xprofile 在 GTK / QT 应用中启用输入法。

```bash
tee ~/.xprofile >> /dev/null <<EOF
#fcitx
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
EOF
```

安装 gnome-tweaks 将 fcitx5 设置为开机启动。

```bash
sudo apt install gnome-tweaks 
gnome-tweaks
# 在 Startup Applications 中选择 fcitx5
# 创建的启动项位于 ~/.config/autostart/org.fcitx.Fcitx5.desktop
```

#### 3.2.4 完全卸载并清理 Snap 服务

如果无需使用 Snap 服务，可按如下方法完全卸载。

先卸载已安装的 snap 包：

```bash
sudo snap list

# 假设已安装下面的包，使用下面的命令卸载
sudo snap remove --purge firefox
sudo snap remove --purge thunderbird
sudo snap remove --purge snap-store
sudo snap remove --purge gnome-42-2204
sudo snap remove --purge gtk-common-themes
sudo snap remove --purge snapd-desktop-integration
sudo snap remove --purge bare
sudo snap remove --purge firmware-updater
sudo snap remove --purge canonical-livepatch
sudo snap remove --purge core22
sudo snap remove --purge snapd
sudo apt purge snapd
sudo apt autoremove
```

彻底禁止 snapd 被再次安装：

```bash
sudo tee /etc/apt/preferences.d/no-snap > /dev/null <<EOF
Package: snapd
Pin: release a=*
Pin-Priority: -10
EOF
```

清理可能残留的 Snap 目录或文件：

```bash
rm -rf ~/snap/
rm -rf ~/.snap/
sudo rm -rf /snap/
sudo rm -rf /var/snap
sudo rm -rf /var/cache/snapd/
sudo rm -rf /var/lib/snapd/
```

（以上）
