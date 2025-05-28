# 主题：WSL 命令行管理总结

**Revision:** 1.0  
**最后更新:** 2025/05/16

## 说明

在 Windows Subsystem for Linux (WSL) 中，可以通过 wsl 命令直接管理 WSL 实例。  
本文总结了一些 ​​WSL 命令行常见参数及用法​。
​​
## 1. 基本 WSL 命令​​
​​
启动默认的 WSL 发行版​​：
```powershell
wsl
```

指定用户名启动：
```powershell
# wsl --user <用户名> 或简写：wsl -u <用户名>
wsl -u root  
```

​​列出已安装的发行版​​：
```powershell
wsl --list --verbose
# 或简写：wsl -l -v
```

输出示例：
```txt
  NAME      STATE           VERSION
* Ubuntu    Running         2
  Debian    Stopped         1
```

​​设置默认发行版​​：
```powershell
# wsl --set-default <发行版名称>
wsl --set-default Debian
```

​​设置默认 WSL 版本（1 或 2）​​：
```powershell
# 设置后，新安装的发行版将默认使用版本 2
wsl --set-default-version 2
```

## ​2. 管理 WSL 发行版​​

​​启动指定发行版​​：
```powershell
# wsl -d <发行版名称>
wsl -d Debian
```

​​停止正在运行的发行版​​​​：
```powershell
# wsl --terminate <发行版名称>
wsl --terminate Ubuntu
```

​​关闭所有 WSL 发行版（相当于重启 WSL 子系统）​​​：
```powershell
wsl --shutdown
```


​​卸载 WSL 发行版​​：
```powershell
# wsl --unregister <发行版名称>
wsl --unregister Debian
```
**​注意​​：** 此操作会删除所有数据，需谨慎操作！

## ​​3. 运行特定命令（不进入 Shell）​​
​​在 WSL 中执行命令并退出​​：
```powershell
# wsl <命令>
wsl ls -l /home
wsl echo "Hello from WSL"
```

​​以 root 身份运行命令​​：
```powershell
# wsl -u root <命令>
wsl -u root apt update
```

## ​4. 备份及导入 WSL 发行版​​
​​
导出 WSL 发行版为备份文件​​：
```powershell
# wsl --export <发行版名称> <备份文件路径>
wsl --export Ubuntu C:\backup\ubuntu_bak.tar
```

​​从备份文件导入 WSL 发行版​​​：
```powershell
#wsl --import <新发行版名称> <安装路径> <备份文件路径>
wsl --import Ubuntu C:\WSL\Ubuntu C:\backup\ubuntu_bak.tar
```

## 5. 其他

​​更改 WSL 发行版版本（转换可能需要几分钟）：
```powershell
# wsl --set-version <发行版名称> <版本号>
wsl --set-version Ubuntu 2
```


​​查看 WSL 版本信息（显示 WSL 版本、内核版本等信息）​​：
```powershell
wsl --version
```

​​在默认目录外启动 WSL​​
```powershell
# wsl --cd <目录路径>，如 Windows 系统本地 C 盘：
wsl --cd /mnt/c
```

## 6. ​​总结​​

| 命令                      | 用途             |
| :------------------------ | :--------------- |
| wsl                       | 启动默认发行版   |
| wsl -l -v                 | 列出所有发行版   |
| wsl --shutdown            | 关闭所有 WSL     |
| wsl --terminate <发行版>  | 停止指定发行版   |
| wsl --unregister <发行版> | 卸载发行版       |
| wsl --export / --import   | 备份/恢复发行版  |
| wsl --set-version         | 切换 WSL 1/2     |
| wsl -u root               | 以 root 身份运行 |

如需要更高级的功能，可以查阅 [Microsoft WSL 官方文档](https://learn.microsoft.com/zh-cn/windows/wsl/)。

或运行 `wsl --help` 命令查看帮助：

```txt
PS C:\Users\Admin> wsl --help
版权所有 (c) Microsoft Corporation。保留所有权利。
有关此产品的隐私信息，请访问 https://aka.ms/privacy。

用法: wsl.exe [Argument][Options...][CommandLine]

运行 Linux 二进制文件的参数:

    如果未提供命令行，wsl.exe 将启动默认 shell。

    --exec, -e <CommandLine>
        在不使用默认 Linux shell 的情况下执行指定的命令。

    --shell-type <standard|login|none>
        使用提供的 shell 类型执行指定的命令。

    --
        按原样传递剩余的命令行。

选项:
    --cd <Directory>
        将指定目录设置为当前工作目录。
        如果使用 ~，则将使用 Linux 用户的主路径。如果路径以
        / 字符开始，它将被解释为绝对 Linux 路径。
        否则，该值必须是绝对 Windows 路径。

    --distribution, -d <DistroName>
        运行指定的分发版。

    --distribution-id <DistroGuid>
        运行指定的分发版 ID。

    --user, -u <UserName>
        以指定用户身份运行。

    --system
        为系统分发版启动 shell。

用于管理适用于 Linux 的 Windows 子系统的参数:

    --help
        显示使用情况信息。

    --debug-shell
        出于诊断目的打开 WSL2 调试 shell。

    --install [Distro] [Options...]
        安装适用于 Linux 的 Windows 子系统分发版。
        有关有效分发版的列表，请使用 'wsl.exe --list --online'。

        选项:
            --enable-wsl1
                启用 WSL1 支持。

            --fixed-vhd
                创建固定大小的磁盘来存储分发版。

            --from-file <Path>
                从本地文件安装分发版。

            --legacy
                使用旧分发版清单。

            --location <Location>
                设置分发版的安装路径。

            --name <Name>
                设置分发版的名称。

            --no-distribution
                仅安装所需的可选组件，不安装分发版。

            --no-launch, -n
                安装后不要启动分发版。

            --version <Version>
                指定要用于新分发的版本。

            --vhd-size <MemoryString>
                指定用于存储分发版的磁盘的大小。

            --web-download
                从 Internet 而不是 Microsoft Store 下载分发版。

    --manage <Distro> <Options...>
        更改发行版特定选项。

        选项:
            --move <Location>
                将分发版移到新位置。

            --set-sparse, -s <true|false>
                将发行版的 vhdx 设置为稀疏，从而允许自动回收磁盘空间。

            --set-default-user <Username>
                设置分发版的默认用户。

            --resize <MemoryString>
                将分发版的磁盘大小调整为指定大小。

    --mount <Disk>
        在所有 WSL 2 分发版中附加和装载物理磁盘或虚拟磁盘。

        选项:
            --vhd
                指定 <Disk> 引用虚拟硬盘。

            --bare
                将磁盘附加到 WSL2，但不要装载它。

            --name <Name>
                使用装入点的自定义名称装载磁盘。

            --type <Type>
                装载磁盘时要使用的文件系统(如果未指定)默认为 ext4。

            --options <Options>
                其他装载选项。

            --partition <Index>
                要装载的分区的索引(如果未指定)默认为整个磁盘。

    --set-default-version <Version>
        更改新分发版的默认安装版本。

    --shutdown
        立即终止所有正在运行的分发版和 WSL 2
        轻型实用工具虚拟机。

    --status
        显示适用于 Linux 的 Windows 子系统状态。

    --unmount [磁盘]
        从所有 WSL2 分发版中卸载和分离磁盘。
        如果在没有参数的情况下调用，则卸载和分离所有磁盘。

    --uninstall
        从此计算机卸载适用于 Linux 的 Windows 子系统包。

    --update
        更新适用于 Linux 的 Windows 子系统包。

        选项:
            --pre-release
                下载预发行版本(如果可用)。

    --version, -v
        显示版本信息。

用于在适用于 Linux 的 Windows 子系统中管理分发版的参数:

    --export <Distro> <FileName> [选项]
        将分发版导出到 tar 文件。
        文件名可以是 - for stdout。

        选项:
            --format <Format>
                指定导出格式。支持的值: tar、tar.gz、tar.xz、vhd。

    --import <Distro> <InstallLocation> <FileName> [选项]
        将指定的 tar 文件作为新分发版导入。
        文件名可以是 - for stdin。

        选项:
            --version <Version>
                指定要用于新分发的版本。

            --vhd
                指定所提供的文件是 .vhdx 文件，而不是 tar 文件。
                此操作在指定的安装位置创建 .vhdx 文件的副本。

    --import-in-place <Distro> <FileName>
        将指定的 .vhdx 文件作为新分发版导入。
        必须使用 ext4 文件系统类型设置此虚拟硬盘的格式。

    --list, -l [选项]
        列出分发版。

        选项:
            --all
                列出所有分发版，包括当前
                正在安装或卸载的分发版。

            --running
                仅列出当前正在运行的分发版。

            --quiet, -q
                仅显示分发版名称。

            --verbose, -v
                显示有关所有分发版的详细信息。

            --online, -o
                显示适合通过 'wsl.exe --install' 安装的可用分发版列表。

    --set-default, -s <Distro>
        将分布版设置为默认值。

    --set-version <Distro> <Version>
        更改指定分发版的版本。

    --terminate, -t <Distro>
        终止指定的分发版。

    --unregister <Distro>
        取消注册分发版并删除根文件系统。
```

(以上)