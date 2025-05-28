# 主题：WSL 中使用 Arch Linux

**Revision:** 1.0  
**最后更新:** 2025/05/16

- Wiki: https://wiki.archlinux.org/title/Install_Arch_Linux_on_WSL

## 1. 升级到最新预览版 

在 Windows 命令提示符运行：
```powershell
wsl --update --pre-release
```

## 2. 安装到指定目录

在 Windows 命令提示符运行：
```powershell
wsl --install -d archlinux --name Arch --location C:\WSL\Arch
```

## 3. 设置用户名及密码

运行 WSL 后，默认进入 root 的 shell。先设置 root 用户的密码，并新建一个用户。

```sh
# 设置 root 用户的密码
passwd root

# 添加一个 arch，useradd 使用 -m 参数同时创建家目录 
useradd -m arch 

# 设置 arch 用户的密码
passwd arch
```

## 4. 设置 pacman 包管理器

pacman 的主要文件
- 主配置文件: /etc/pacman.conf
- 镜像列表: /etc/pacman.d/mirrorlist

```sh
# 备份镜像列表
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak

# 添加国内镜像源（nju）
sed -i '1i\Server = https://mirrors.nju.edu.cn/archlinux/$repo/os/$arch' /etc/pacman.d/mirrorlist

# 更新软件包数据库 
pacman -Syy

# 全面更新系统（可选）
pacman -Syu

# 使用 reflector 自动选择最佳镜像（可选）
pacman -S reflector
reflector --country China --protocol https --sort rate --save /etc/pacman.d/mirrorlist

# 安装 sudo 等必备工具
pacman -S sudo nano which util-linux
```

## 5. 使用 arch 用户

### 5.1. 为 arch 用户添加 sudo 权限

```sh
# 将 arch 用户添加到 wheel 用户组以使用 sudo 
usermod -aG wheel arch

# 启用 wheel 用于组的 sudo 权限
sed -i '/^#\s*%wheel\s*ALL=(ALL:ALL)\s*ALL/s/^#\s*//' /etc/sudoers
```

切换到 arch 用户，验证 sudo 可用：
```sh
su - arch
sudo whoami
# 应显示为 root
```

### 5.2. 将 WSL 启动时的默认用户修改为 arch

在 /etc/wsl.conf 中追加如下内容：
```txt
[user]
default=arch
```

在 Windows 命令提示符下关闭并重启 WSL2 验证效果，再次启动后，用户默认应为 arch：
```powershell
wsl --terminate Arch
```

### 5.3 为 nano 添加语法着色

运行如下命令：
```sh
echo 'include "/usr/share/nano/*.nanorc"' > ~/.nanorc
# 运行 nano ~/.nanorc 确认有语法着色显示
```

### 5.4 调整 bash 提示符着色

运行 `nano ~/.bashrc`  将 PS1 变量修改为如下内容：
```
#PS1='[\u@\h \W]\$ '
PS1='\[\e[1;32m\]\u@\h\[\e[0m\]:\[\e[1;34m\]\w\[\e[0m\]\$ '
```
然后运行 `source ~/.bashrc` 启用配置，bash 用户名部分会变成绿色。

※或者也可以使用 oh-my-bash 等开源项目配置。

## 6. 用 WSLg 运行图形界面应用程序 (可选)

### 6.1. 使用 WSLg 准备步骤

需要 WSL2 版本在 2.5.7.0 或以上。  
在 Windows 系统的 %USERPROFILE%\.wslconfig 文件中启用对图形界面应用程序的支持：
```txt
[wsl2]
guiApplications = true
```

创建 /etc/profile.d/wslg.sh 文件，内容如下：
```sh
export GALLIUM_DRIVER=d3d12

for i in "/mnt/wslg/runtime-dir/"*; do
  [ "$XDG_RUNTIME_DIR" = "$HOME" ] && XDG_RUNTIME_DIR="/var/run/user/$UID"
  if [ ! -L "$XDG_RUNTIME_DIR$(basename "$i")" ]; then
    [ -d "$XDG_RUNTIME_DIR$(basename "$i")" ] && rm -r "$XDG_RUNTIME_DIR$(basename "$i")"
    ln -s "$i" "$XDG_RUNTIME_DIR$(basename "$i")"
  fi
done
```

设置硬件加速渲染：
```sh
# 安装硬件减速驱动包：
sudo pacman -S mesa vulkan-dzn vulkan-icd-loader

# 若 openGL 依然在英特尔 GPU 上使用 llvmpipe 软件渲染，则需要为 libedit 创建符号链接：
sudo ln -s /usr/lib/libedit.so /usr/lib/libedit.so.2
```

在 Windows 命令提示符下运行  `wsl --terminate Arch` 后，重新启动 WSL 就能使用 WSLg 了。

### 6.2. 安装运行 WSLg 程序

以 gedit 为例：
```sh
# 安装 gedit
sudo pacman -S gedit

# 运行 gedit
gedit &
```

安装小企鹅输入法 fcitx5：
```sh
# 安装 小企鹅输入法 fcitx5
sudo pacman -S fcitx5 fcitx5-configtool fcitx5-gtk fcitx5-qt fcitx5-chinese-addons

# 安装字体支持
sudo pacman -S noto-fonts-cjk noto-fonts-emoji
```

先手动运行 `fcitx5 --disable=wayland -d` 命令后，运行 `fcitx5-configtool` 进行如下设置：
- Input Method 标签页：添加 Pinyin 输入法。
- Addons 标签页: 勾上 Show advanced options 搜索 Wayland，将 Wayland前的勾去掉。

在 WSLg 应用（如 gedit）中 ，按快捷键 Ctrl + Space 可切换到中文拼音输入法。

设置 WSL 启动时自动运行 fcitx5。在 ~/.bashrc 最后加上如下内容：
```sh
#fcitx5
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
# fcitx5 --disable=wayland -d &> /dev/null &
fcitx5 -d &> /dev/null &
```

## 7. Pacman 命令总结

### 7.1. 安装与升级

| 命令 | 说明 |
|------|------|
| `sudo pacman -S <包名>` | 安装指定软件包 |
| `sudo pacman -Sy` | 刷新软件包数据库（同步远程仓库） |
| `sudo pacman -Syu` | **升级系统**（刷新数据库 + 升级所有包） |
| `sudo pacman -U <本地包路径.pkg.tar.zst>` | 从本地文件安装软件包 |
| `sudo pacman -S <包名> --needed` | 安装时跳过已安装的最新版本 |

### 7.2. 卸载

| 命令 | 说明 |
|------|------|
| `sudo pacman -R <包名>` | 卸载指定软件包（保留依赖） |
| `sudo pacman -Rs <包名>` | 卸载软件包及**未被其他包依赖的依赖项** |
| `sudo pacman -Rsc <包名>` | 卸载软件包及**所有依赖项**（慎用！可能破坏系统） |
| `sudo pacman -Rns <包名>` | 卸载软件包并删除相关配置文件（`-n` 清理配置） |

### 7.3. 查询与搜索

| 命令 | 说明 |
|------|------|
| `pacman -Ss <关键词>` | 在仓库中搜索包含关键词的软件包 |
| `pacman -Qs <关键词>` | 在**已安装的包**中搜索 |
| `pacman -Qi <包名>` | 查看软件包详细信息（版本、依赖等） |
| `pacman -Ql <包名>` | 列出软件包安装的所有文件 |
| `pacman -Qdt` | 列出**孤立包**（无依赖的包） |
| `pacman -Qm` | 列出手动安装的非仓库包（如 AUR 包） |

### 7.4. 维护与清理

| 命令 | 说明 |
|------|------|
| `sudo pacman -Sc` | 清理**未安装的旧版本包缓存** |
| `sudo pacman -Scc` | 清理**所有包缓存**（慎用！可能导致降级困难） |
| `sudo pacman -D --asexplicit <包名>` | 将包标记为“显式安装” |
| `sudo pacman -D --asdeps <包名>` | 将包标记为“依赖项” |

### 7.5. 高级操作

| 命令 | 说明 |
|------|------|
| `sudo pacman -Syyu` | 强制刷新数据库并升级（修复数据库不一致） |
| `sudo pacman -S <包名> --overwrite '*'` | 强制覆盖冲突文件（谨慎使用！） |
| `sudo pacman -Fy` | 更新文件数据库（用于 `pacman -F` 文件搜索） |
| `sudo pacman -F <文件名>` | 查找哪个包包含指定文件 |

### 7.6. 常见问题处理

1. **依赖冲突**：  
   - 尝试 `sudo pacman -Syu` 更新系统。  
   - 若仍失败，可手动移除冲突包（谨慎操作）。

2. **清理孤立包**：  
```bash
sudo pacman -Qdtq | sudo pacman -Rs -
```

3. **强制降级包**：  
```bash
sudo pacman -U /var/cache/pacman/pkg/<旧版本包名>.pkg.tar.zst
```

注意事项：
- 定期运行 sudo pacman -Syu 保持系统更新。
- 谨慎使用 -Rsc 或 --force 参数，可能导致系统不稳定。
- 重要操作前建议备份配置文件（如 /etc 目录）。

Wiki: https://wiki.archlinuxcn.org/wiki/Pacman

（以上）