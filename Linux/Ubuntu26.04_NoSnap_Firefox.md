# 主题：Ubuntu 26.04 删除 Snap 并为 Firefox 添加 APT 源

**Revision:** 1.0  
**最后更新:** 2026/11/10

## 1. 列出已安装的 snap 包

Ubuntu 26.04 运行 snap list 结果示例如下：

```
snap list
Name                       Version                         Rev    Tracking         Publisher            Notes
bare                       1.0                             5      latest/stable    canonical✓           base
core24                     20260824                        2124   latest/stable    canonical✓           base
desktop-security-center    0+git.4adf2b9                   206    1/stable/…       canonical✓           -
firefox                    159.0.1-1                       9171   latest/stable/…  mozilla✓             -
firmware-updater           0+git.64c41a0                   262    1/stable/…       canonical✓           -
gnome-46-2404              0+git.b31ceab-sdk0+git.ef6f7e9  177    latest/stable/…  canonical✓           -
gtk-common-themes          0.1-81-g442e511                 1535   latest/stable/…  canonical✓           -
hwctl                      0.11.1                          131    latest/stable/…  ce-certification-qa  -
mesa-2404                  25.2.8-snap288                  1839   latest/stable/…  canonical✓           -
prompting-client           0+git.dd8c750                   228    1/stable/…       canonical✓           -
snap-store                 0+git.5534399d                  1468   2/stable/…       canonical✓           -
snapd                      2.78                            28332  latest/stable    canonical✓           snapd
snapd-desktop-integration  0.9                             391    latest/stable/…  canonical✓           -
```

## 2. 删除所有的 snap 包

将脚本保存为 `remove_snap_apps.sh` 并使用 sudo 运行。

```bash
#!/bin/bash

set -u

if ! command -v snap >/dev/null 2>&1; then
    echo "snap command not found."
    exit 1
fi

sudo -v || exit 1

# Remove application snaps first, followed by runtime dependencies.
SNAPS="
firefox
snap-store
desktop-security-center
firmware-updater
hwctl
prompting-client
snapd-desktop-integration
gtk-common-themes
gnome-46-2404
mesa-2404
core24
bare
"

# Retry failed removals in case of dependency conflicts.
nMaxRounds=5

for ((nRound = 1; nRound <= nMaxRounds; nRound++)); do
    bPending=0
    bRemoved=0

    echo
    echo "========================================"
    echo "Removal round: $nRound"
    echo "========================================"

    for strSnap in $SNAPS; do
        if ! sudo snap list "$strSnap" >/dev/null 2>&1; then
            continue
        fi

        echo "Removing: $strSnap"

        if sudo snap remove --purge "$strSnap"; then
            bRemoved=1
        else
            echo "WARNING: Failed to remove $strSnap"
            bPending=1
        fi
    done

    if [ "$bPending" -eq 0 ]; then
        break
    fi

    if [ "$bRemoved" -eq 0 ]; then
        break
    fi
done

# Verify remaining snaps, excluding the snapd snap itself.
bPending=0

for strSnap in $SNAPS; do
    if sudo snap list "$strSnap" >/dev/null 2>&1; then
        echo "WARNING: Still installed: $strSnap"
        bPending=1
    fi
done

if [ "$bPending" -ne 0 ]; then
    echo "ERROR: Some snaps could not be removed."
    echo "Please check the dependencies and retry."
    exit 1
fi

# Remove the system snapd package.
echo
echo "Purging snapd package..."

if ! sudo apt purge -y snapd; then
    echo "ERROR: Failed to purge snapd."
    exit 1
fi

# Clean up unused dependencies.
sudo apt autoremove --purge -y

echo
echo "========================================"
echo "Snap removal completed."
echo "========================================"
```

## 3. 禁止 snap 被再次安装 

将脚本保存为 `disable_snap_apt.sh` 并使用 sudo 运行。

```bash
#!/bin/bash

set -eu

if [ "$(id -u)" -ne 0 ]; then
    echo "Please run this script as root."
    exit 1
fi

PREF_FILE="/etc/apt/preferences.d/deny-snapd"

echo "========================================"
echo "Disabling Snap installation through APT"
echo "========================================"

# Create APT pinning rules.
cat > "$PREF_FILE" <<'EOF'
Explanation: Prevent installation of Snap daemon packages
Package: snapd snapd-*
Pin: version *
Pin-Priority: -1

Explanation: Prevent installation of binaries built from snapd source
Package: src:snapd
Pin: version *
Pin-Priority: -1
EOF

echo "Created: $PREF_FILE"

# Refresh APT package information.
apt-get update

# Verify the package policy.
echo
echo "========================================"
echo "APT policy for snapd"
echo "========================================"

apt-cache policy snapd

echo
echo "Snap installation blocking rules configured."
```

## 4. 添加 Mozilla 官方 APT 源

将脚本保存为 `setup_mozilla_repo.sh` 并使用 sudo 运行。

**注意：**
- `wget` 可能需预先配置代理，具体参照**代理服务器配置**的相关文档。
- 添加完成后，使用 `sudo apt install firefox` 或 `sudo apt install firefox-esr` 安装。

```bash
#!/bin/bash

set -euo pipefail

if [ "$(id -u)" -ne 0 ]; then
    echo "Please run this script as root."
    exit 1
fi

if ! command -v wget >/dev/null 2>&1 || ! command -v gpg >/dev/null 2>&1; then
    echo "ERROR: wget and gpg are required."
    exit 1
fi

KEY_FILE="/etc/apt/keyrings/packages.mozilla.org.asc"
SOURCE_FILE="/etc/apt/sources.list.d/mozilla.sources"
PREF_FILE="/etc/apt/preferences.d/mozilla"

EXPECTED_FINGERPRINT="35BAA0B33E9EB396F59CA838C0BA5CE6DC6315A3"

TMP_KEY=$(mktemp)
trap 'rm -f "$TMP_KEY"' EXIT

echo "========================================"
echo "Configuring Mozilla APT repository"
echo "========================================"

# Download the Mozilla signing key.
wget -q https://packages.mozilla.org/apt/repo-signing-key.gpg \
    -O "$TMP_KEY"

# Verify the signing key fingerprint.
FINGERPRINT=$(gpg --show-keys --with-colons "$TMP_KEY" |
    awk -F: '$1 == "fpr" { print $10; exit }')

if [ "$FINGERPRINT" != "$EXPECTED_FINGERPRINT" ]; then
    echo "ERROR: Mozilla signing key fingerprint mismatch."
    echo "Expected: $EXPECTED_FINGERPRINT"
    echo "Actual:   $FINGERPRINT"
    exit 1
fi

echo "Mozilla signing key verified."

# Install the signing key.
install -d -m 0755 /etc/apt/keyrings
install -m 0644 "$TMP_KEY" "$KEY_FILE"

# Configure the Mozilla repository.
cat > "$SOURCE_FILE" <<EOF
Types: deb
URIs: https://packages.mozilla.org/apt
Suites: mozilla
Components: main
Signed-By: $KEY_FILE
EOF

# Configure APT package priorities.
cat > "$PREF_FILE" <<'EOF'
Explanation: Prefer packages from the official Mozilla repository
Package: *
Pin: origin packages.mozilla.org
Pin-Priority: 1000

Explanation: Prevent Ubuntu Firefox transitional Snap package
Package: firefox
Pin: release o=Ubuntu
Pin-Priority: -1
EOF

echo "Mozilla repository configured."

# Update package indexes.
apt-get update

echo
echo "========================================"
echo "Mozilla APT repository setup completed."
echo "========================================"

echo
echo "Firefox package policy:"
apt-cache policy firefox
```

## 5. 删除不必要的硬件驱动包（VMware环境）

VMware环境中，可删除一些不必要的硬件驱动包（以 Intel 主机为例）。

```
sudo apt-get purge \
    linux-firmware \
    firmware-sof-signed \
    linux-firmware-amd-graphics \
    linux-firmware-amd-misc \
    linux-firmware-broadcom-wireless \
    linux-firmware-intel-graphics \
    linux-firmware-intel-wireless \
    linux-firmware-marvell-prestera \
    linux-firmware-marvell-wireless \
    linux-firmware-mediatek \
    linux-firmware-mellanox-spectrum \
    linux-firmware-netronome \
    linux-firmware-nvidia-graphics \
    linux-firmware-qlogic \
    linux-firmware-qualcomm-graphics \
    linux-firmware-qualcomm-misc \
    linux-firmware-qualcomm-wireless
```

(以上)