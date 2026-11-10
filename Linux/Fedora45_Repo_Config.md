# 主题：Fedora 镜像源配置

**Revision:** 1.0  
**最后更新:** 2026/11/10


**说明：**
- 本文适用于 Fedora 45 配置，后续版本也可供参考。


## 1. Beta版 镜像源配置

DNF5 默认源文件保持原样：
- `/usr/share/dnf5/repos.d/fedora.repo`
- `/usr/share/dnf5/repos.d/fedora-updates.repo`

修改或创建 `/etc/dnf/repos.override.d/99-config_manager.repo`，内容如下：

```ini
[fedora]
metalink=
baseurl=https://mirrors.nju.edu.cn/fedora/releases/test/${releasever}_Beta/Everything/$basearch/os/
[updates]
metalink=
baseurl=https://mirrors.nju.edu.cn/fedora/updates/$releasever/Everything/$basearch/
[updates-testing]
enabled=0
[fedora-cisco-openh264]
enabled=0
```

## 2. 正式版 镜像源配置

DNF5 默认源文件保持原样：
- `/usr/share/dnf5/repos.d/fedora.repo`
- `/usr/share/dnf5/repos.d/fedora-updates.repo`

修改或创建 `/etc/dnf/repos.override.d/99-config_manager.repo`，内容如下：

```ini
[fedora]
metalink=
baseurl=https://mirrors.nju.edu.cn/fedora/releases/$releasever/Everything/$basearch/os/
[updates]
metalink=
baseurl=https://mirrors.nju.edu.cn/fedora/updates/$releasever/Everything/$basearch/
[updates-testing]
enabled=0
[fedora-cisco-openh264]
enabled=0
```


## 3. Beta版 升级到 正式版

将 `/etc/dnf/repos.override.d/99-config_manager.repo` 改为正式版配置，然后运行如下命令：

```bash
sudo dnf clean all
sudo dnf makecache
sudo dnf update
```

## 4. 删除未使用的驱动

运行如下脚本命令，可查看未使用的驱动：

```bash
sudo bash <<'EOF'
DMESG=$(dmesg 2>/dev/null)
rpm -qa | grep -iE "firmware" | grep -v "linux-firmware-whence" | sort | while read pkg; do
  files=$(rpm -ql "$pkg" 2>/dev/null \
    | grep "^/usr/lib/firmware/" \
    | sed 's#.*/##; s#\.xz$##; s#\.bin$##' | sort -u)
  [ -z "$files" ] && continue
  used=0
  for f in $files; do
    if echo "$DMESG" | grep -qiE "$f"; then used=1; break; fi
  done
  [ "$used" -eq 0 ] && echo "UNUSED: $pkg"
done
EOF
```

确认后可运行 `sudo dnf remove xxx` 删除不要的驱动包。

## 5. BTRFS Snapper 配置

Fedora 默认使用 BTRFS 文件系统，可安装相关工具进行 Snapper 配置。

```bash
sudo dnf install snapper btrfs-assistant libdnf5-plugin-actions

sudo tee /etc/dnf/libdnf5-plugins/actions.d/snapper.actions >/dev/null << 'EOF'
# /etc/dnf/libdnf5-plugins/actions.d/snapper.actions

pre_transaction::::/usr/bin/sh -c echo\ "tmp.cmd=$(ps\ -o\ command\ --no-headers\ -p\ '${pid}')"
pre_transaction::::/usr/bin/sh -c echo\ "tmp.snapper_pre_number=$(snapper\ create\ -c\ number\ -t\ pre\ -p\ -d\ '${tmp.cmd}')"

post_transaction::::/usr/bin/sh -c [\ -n\ "${tmp.snapper_pre_number}"\ ]\ &&\ snapper\ create\ -c\ number\ -t\ post\ --pre-number\ "${tmp.snapper_pre_number}"\ -d\ "${tmp.cmd}"\ ;\ echo\ tmp.snapper_pre_number\ ;\ echo\ tmp.cmd
EOF
```

（以上）