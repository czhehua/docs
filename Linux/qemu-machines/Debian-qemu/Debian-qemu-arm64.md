# Debian amd64 qemu

## 1. Host OS Environment

- WSL2 Debian 13, or
- Debian 13 Trixie amd64

Install the following packages:
```bash
sudo apt update
sudo apt install qemu-utils qemu-system-arm qemu-efi-aarch64
```

## 1. Get qcow2 image

Download official Debian cloud image from:
- https://cdimage.debian.org/cdimage/cloud/

Or download from a mirror site:
- http://mirrors.ustc.edu.cn/debian-cdimage/cloud/
- http://mirrors.nju.edu.cn/debian-cdimage/cloud/

For example, Debian 13 Trixie qcow2 image, download the following image:
- https://cdimage.debian.org/cdimage/cloud/trixie/latest/debian-13-nocloud-arm64.qcow2

Move the downloaded image to a new folder:
```bash
mkdir qemu-debian-13-arm64
mv debian-13-nocloud-arm64.qcow2 qemu-debian-13-arm64/
```

## 2. Prepare the image and startup script

Verify and adjust the qcow2 disk size:
```bash
# Check qcow2 image virtual size (default only 3 GB)
qemu-img info debian-13-nocloud-arm64.qcow2

# Resize qcow2 image virtual size to 20 GB
qemu-img resize debian-13-nocloud-arm64.qcow2 20G
```

Create a start shell for qcow2 image.

```bash
tee start.sh > /dev/null <<'EOF'
#!/bin/sh

qemu-system-aarch64 \
    -m 2G \
    -cpu cortex-a72 \
    -smp 2 \
    -M virt \
    -bios /usr/share/qemu-efi-aarch64/QEMU_EFI.fd \
    -drive if=none,file=debian-13-nocloud-arm64.qcow2,id=hd0 \
    -device virtio-blk-device,drive=hd0 \
    -netdev user,id=net0,hostfwd=tcp::2222-:22 \
    -device virtio-net-device,netdev=net0 \
    -nographic
EOF

chmod +x start.sh
```

## 3. Start the qemu machine

Start the qemu machine with the shell:
```bash
./start.sh
```

Login with user `root`, run the following command to enlarge root size:
```bash
lsblk
apt update && apt install -y cloud-guest-utils
growpart /dev/vda 1
resize2fs /dev/vda1
lsblk
```

Now the root size should be 19.9G. Example:
```txt
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
vda     254:0    0   20G  0 disk
├─vda1  254:1    0 19.9G  0 part /
└─vda15 254:15   0  127M  0 part /boot/efi
```

Note: qmeu machine can connected via port 2222 after openssh-server is configured.

## 4. Boot with GUI

In case GUI has been installed, change `start.sh` like following.

```bash
qemu-system-aarch64 \
  -m 2G \
  -cpu cortex-a72 \
  -smp 2 \
  -M virt \
  -bios /usr/share/qemu-efi-aarch64/QEMU_EFI.fd \
  -drive if=none,file=debian-12-nocloud-arm64.qcow2,id=hd0 \
  -device virtio-blk-device,drive=hd0 \
  -netdev user,id=net0,hostfwd=tcp::2223-:22 \
  -device virtio-net-device,netdev=net0 \
  -device virtio-gpu-pci -display gtk,gl=on \
  -device usb-ehci,id=ehci \
  -device usb-kbd \
  -device usb-tablet \
  -serial mon:stdio
```

(END)