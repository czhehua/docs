# Fedora 45 配置

## Fedora 45 Beta版 镜像源配置

DNF5 默认源文件保持原样：
- `/usr/share/dnf5/repos.d/fedora.repo`
- `/usr/share/dnf5/repos.d/fedora-updates.repo`

修改或创建 `/etc/dnf/repos.override.d/99-config_manager.repo`，内容如下：

```txt
[fedora]
metalink=
baseurl=https://mirrors.nju.edu.cn/fedora/releases/test/45_Beta/Everything/$basearch/os/
[updates]
metalink=
baseurl=https://mirrors.nju.edu.cn/fedora/updates/$releasever/Everything/$basearch/
[updates-testing]
enabled=0
[fedora-cisco-openh264]
enabled=0
```

## Fedora 45 正式版 镜像源配置

DNF5 默认源文件保持原样：
- `/usr/share/dnf5/repos.d/fedora.repo`
- `/usr/share/dnf5/repos.d/fedora-updates.repo`

修改或创建 `/etc/dnf/repos.override.d/99-config_manager.repo`，内容如下：

```txt
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


## Fedora 45 Beta版 升级到 Fedora 45 正式版

将 `/etc/dnf/repos.override.d/99-config_manager.repo` 改为正式版配置，然后运行如下命令：

```bash
sudo dnf clean all
sudo dnf makecache
sudo dnf update
```

## Fedora 删除没用到的驱动

运行如下脚本，可查看没用到的驱动：

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