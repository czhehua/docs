# 主题：WSL 使用入门

**Revision:** 1.0  
**最后更新:** 2025/05/16

## 1. 启用 WSL 功能

**系统：**
- Windows 10 2004以后 (x64)
- Windows 11 (x64)

**方法：**
- 运行 `appwiz.cpl` 在 "程序和功能" 对话框左侧 点击 "启用或关闭 Windows 功能"（或Win键+R 直接运行 `optionalfeatures` 命令打开），勾选最下面的 “适用于 Linux 的 Windows 子系统” 后，点确定。

在 Hyper-V 客户中使用WSL，需先打开嵌套虚拟化：
```
Set-VMProcessor -VMName <虚拟机名称> -ExposeVirtualizationExtensions $true
```

## 2. 升级 WSL

运行命令升级到最新 WSL 版本：
```powershell
wsl --update
```

或升级到最新 WSL 预览版本（含有实验性的新功能）：
 ```powershell
wsl --update --pre-release
```

更新后若提示重启，需按提示重新启动 Windows 系统。

## 3. 列出支持的发行版

运行命令：
```powershell
wsl --list --online
```
会列出当前支持的发行版（例如2025年5月返回如下信息）：
```txt
以下是可安装的有效分发的列表。
使用 'wsl.exe --install <Distro>' 安装。

NAME                            FRIENDLY NAME
AlmaLinux-8                     AlmaLinux OS 8
AlmaLinux-9                     AlmaLinux OS 9
AlmaLinux-Kitten-10             AlmaLinux OS Kitten 10
Debian                          Debian GNU/Linux
FedoraLinux-42                  Fedora Linux 42
SUSE-Linux-Enterprise-15-SP5    SUSE Linux Enterprise 15 SP5
SUSE-Linux-Enterprise-15-SP6    SUSE Linux Enterprise 15 SP6
Ubuntu                          Ubuntu
Ubuntu-24.04                    Ubuntu 24.04 LTS
archlinux                       Arch Linux
kali-linux                      Kali Linux Rolling
openSUSE-Tumbleweed             openSUSE Tumbleweed
openSUSE-Leap-15.6              openSUSE Leap 15.6
Ubuntu-18.04                    Ubuntu 18.04 LTS
Ubuntu-20.04                    Ubuntu 20.04 LTS
Ubuntu-22.04                    Ubuntu 22.04 LTS
OracleLinux_7_9                 Oracle Linux 7.9
OracleLinux_8_7                 Oracle Linux 8.7
OracleLinux_9_1                 Oracle Linux 9.1
```

## 4. 选择一个发行版安装

以 Debian 为例，运行如下命令安装：
```powershell
wsl --install Debian
```

通过参数可自定义名称和安装位置，例如：
```powershell
# 通过 --name 指定名称 --location 指定安装目录
wsl --install Debian --name <自定义名称> --location C:\WSL\Debian
```
安装目录中将会有 `ext4.vhdx` 和 `shortcut.ico` 两个文件。  

运行 WSL2 发行版，相当于运行一台 Hyper-V 客户机。

## 5. 开始运行

安装完成后，可启动 WSL：
```powershell
wsl -d <名称>
```
首次启动时，根据发行版不同，有些需要设置用户名和密码。

## 6. 使用 WSL

与使用 Linux 发行版的终端一样使用 WSL。

Windows 本地磁盘会被自动挂载到 `/mnt/盘符`，例如：
- C盘： /mnt/c/
- D盘： /mnt/d/

## 7. 管理 已安装的 WSL 发行版

列出本地已安装的发行版：
```powershell
wsl --list --verbose
```
会显示如下的信息：
```txt
  NAME      STATE           VERSION
* Debian    Running         2
  Ubuntu     Stopped        2
```

关闭 WSL 发行版：
```powershell
wsl --terminate <名称>
```

从本地删除一台 WSL 发行版：
```powershell
wsl --unregister <名称>
```

## 8. 备份和恢复

备份 WSL 发行版到 tar 文件。例如：
```powershell
# wsl --export <名称> <tar路径>
wsl --export Debian C:\WSL\debian-backup.tar 
```

拷贝到另一台电脑上，从 tar 恢复 WSL 发行版。例如：
```powershell
# wsl --import <名称> <新的安装路径> <tar路径> --version <WSL版本>
wsl --import Debian C:\WSL\Debian C:\WSL\debian-backup.tar --version 2 
```

有些发行版（如Debian）导入后，需要修改默认用户：​​
- 打开 ​​注册表编辑器​​（regedit）导航至：
  HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Lxss\{发行版GUID}
  修改 DefaultUid 的值为 0（root）或 1000（普通用户）。使用十进制数值。

（以上）