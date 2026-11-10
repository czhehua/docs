# Windows 开启 SSH 服务

## 1. 安装 OpenSSH Server

可按如下步骤开启 Windows 系统的 OpenSSH 服务。PowerShell 命令如下：

```powershell
# 检查 OpenSSH Server
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH.Server*'

# 安装 OpenSSH Server
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# 启动 SSH 服务
Start-Service sshd

# 设置开机启动
Set-Service sshd -StartupType Automatic
```

## 2. 设置防火墙规则

默认状态下，由于防火墙规则，可能无法通过 SSH 访问服务器 22 端口。

查看当前防火墙规则。PowerShell 命令如下：

```powershell
Get-NetFirewallRule -PolicyStore ActiveStore | Where-Object DisplayName -like '*OpenSSH*' | Format-List *
```

默认只对 Private 网络有效，可增加 Public 网络。PowerShell 命令如下：

```powershell
Set-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -Profile Private,Public
```

## 3. 设置默认启动 PowerShell 7

未设置时，SSH 登录时默认启动 CMD。可通过修改注册表，让其默认启动 PowerShell 7。

PowerShell 命令如下：

```powershell
# 设置 SSH 默认 Shell
New-ItemProperty -Path 'HKLM:\SOFTWARE\OpenSSH' `
    -Name DefaultShell `
    -Value 'C:\Program Files\PowerShell\7\pwsh.exe' `
    -PropertyType String -Force

# 重启 SSH 服务
Restart-Service sshd
```

## 4. 连接 SSH 服务器

命令与访问 Linux SSH 服务器一致：

```bash
ssh user@192.168.100.1
```

首次访问时，输入 `yes` 保存服务器信息到 ~/.ssh/known_hosts，输入密码后，即可 SSH 登录服务器。

(以上)