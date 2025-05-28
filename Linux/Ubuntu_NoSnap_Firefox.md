# 主题：Ubuntu 24.04 LTS 替换 Snap 版 Firefox

**Revision:** 1.0  
**最后更新:** 2025/05/13

- [主题：Ubuntu 24.04 LTS 替换 Snap 版 Firefox](#主题ubuntu-2404-lts-替换-snap-版-firefox)
  - [1. 需求](#1-需求)
  - [2. 卸载 Snap 版 Firefox](#2-卸载-snap-版-firefox)
  - [3. 使用 .deb 包安装 Firefox](#3-使用-deb-包安装-firefox)
  - [4. 转移 Firefox 配置文件 (可选)](#4-转移-firefox-配置文件-可选)
  - [5. 禁用软件包自动更新服务 (可选)](#5-禁用软件包自动更新服务-可选)
  - [附录. 错误处理](#附录-错误处理)

## 1. 需求

Ubuntu 24.04 LTS 中默认自带的 Firefox 为 Snap 版，由于其使用沙盒环境，与其他一些软件配合使用时，会有兼容性问题。例如：
- 与输入法不兼容：使用 ibus 或 fcitx 等输入法时，无法切换到中文或日文模式。

安装 Mozilla 官方提供的 deb 版本能解决这些问题。
- 参考 GitHub：[Ubuntu 24.04 Firefox snap replacement](https://gist.github.com/jfeilbach/78d0ef94190fb07dee9ebfc34094702f)


## 2. 卸载 Snap 版 Firefox

运行如下命令进行卸载 Snap 版 Firefox。一般情况下，Snap 版 Firefox 将被移除。如果出错，请参照 [附录-错误处理](#附录-错误处理)。

```bash
sudo snap disable firefox
sudo snap remove --purge firefox
```

输入以下命令，防止 Snap 版 Firefox 被再次安装。

```bash
sudo tee /etc/apt/preferences.d/no-snap-firefox > /dev/null <<EOF
Package: firefox*
Pin: release o=Ubuntu*
Pin-Priority: -1
EOF
```

## 3. 使用 .deb 包安装 Firefox

参考 Mozilla 官方文档：[在 GNU/Linux 中安装 Firefox](https://support.mozilla.org/zh-CN/kb/install-firefox-linux)

1. 创建一个保存 APT 库密钥的目录：

```bash
sudo install -d -m 0755 /etc/apt/keyrings
```

2. 导入 Mozilla APT 密钥环： 

```bash
wget -q https://packages.mozilla.org/apt/repo-signing-key.gpg -O- \
    | sudo tee /etc/apt/keyrings/packages.mozilla.org.asc > /dev/null
```
如果没有安装 wget，请通过命令 sudo apt install wget 安装。 

3. 密钥指纹应该是 `35BAA0B33E9EB396F59CA838C0BA5CE6DC6315A3`。你可以用以下命令检查： 

```bash
gpg -n -q --import --import-options import-show \
    /etc/apt/keyrings/packages.mozilla.org.asc \
    | awk '/pub/ {
        getline; 
        gsub(/^ +| +$/,""); 
        if ($0 == "35BAA0B33E9EB396F59CA838CBA5CE6DC6315A3") 
            print "\nThe key fingerprint matches ("$0").\n"; 
        else 
            print "\nVerification failed: the fingerprint ("$0") does not match the expected one.\n"
      }'
```

4. 把 Mozilla APT 库添加到源列表中： 

```bash
sudo tee -a /etc/apt/sources.list.d/mozilla.list > /dev/null <<EOF
deb [signed-by=/etc/apt/keyrings/packages.mozilla.org.asc] https://packages.mozilla.org/apt mozilla main
EOF
```

5. 配置 APT 优先使用 Mozilla 库中的包： 

```bash
sudo tee /etc/apt/preferences.d/mozilla > /dev/null <<EOF
Package: *
Pin: origin packages.mozilla.org
Pin-Priority: 1000
EOF
```
6. 更新软件列表并安装 firefox（或 firefox-esr、-beta、-nightly、-devedition 之一）： 

```bash
sudo apt update && sudo apt install firefox
```

## 4. 转移 Firefox 配置文件 (可选)

先确认 Snap 版 Firefox 配置文件是否存在。如果不存在，则无需执行后面的操作。

```bash
ls ~/snap/firefox/common/.mozilla/firefox/
```

从 Snap 版 Firefox 迁移至 .deb 版（或其他安装方式）时保留个人数据（书签、扩展、历史记录等）。命令如下：

```
mkdir -p ~/.mozilla/firefox/ && cp -a ~/snap/firefox/common/.mozilla/firefox/* ~/.mozilla/firefox/
```

## 5. 禁用软件包自动更新服务 (可选)

软件包自动更新服务可能会重新安装 Snap 版的 Firefox。建议禁用。

```
sudo systemctl status unattended-upgrades
sudo systemctl disable --now unattended-upgrades
sudo systemctl status unattended-upgrades
```

## 附录. 错误处理

卸载 Snap 版 Firefox 如果出现如下报错信息，可能是系统挂载了与Snap相关的​​只读文件系统​​导致无法卸载。

```txt
error: cannot perform the following tasks:
- Remove data for snap "firefox" (1943) (unlinkat /var/snap/firefox/common/host-hunspell/en_ZA.dic: read-only file system)
```

输入以下命令，解除系统对 /var/snap/firefox 目录的只读挂载状态：

```bash
sudo systemctl stop var-snap-firefox-common-host\\x2dhunspell.mount
sudo systemctl disable var-snap-firefox-common-host\\x2dhunspell.mount
```

若上述步骤后仍无法删除，强制卸载目录：

```bash
sudo umount /var/snap/firefox/common/host-hunspell
```

执行上述命令后，再次尝试 [卸载 Snap 版 Firefox](#2-卸载-snap-版-firefox)。

（以上）
