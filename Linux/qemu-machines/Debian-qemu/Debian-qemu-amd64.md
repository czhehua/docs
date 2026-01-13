# Debian amd64 qemu

## 1. Host OS Environment

- WSL2 Debian 13, or
- Debian 13 Trixie amd64

Install the following packages:
```bash
sudo apt update
sudo apt install qemu-utils qemu-system-x86
```

## 1. Get qcow2 image

Download official Debian cloud image from:
- https://cdimage.debian.org/cdimage/cloud/

Or download from a mirror site:
- http://mirrors.ustc.edu.cn/debian-cdimage/cloud/
- http://mirrors.nju.edu.cn/debian-cdimage/cloud/

For example, Debian 13 Trixie qcow2 image, download the following image:
- https://cdimage.debian.org/cdimage/cloud/trixie/latest/debian-13-nocloud-amd64.qcow2

Move the downloaded image to a new folder:
```bash
mkdir qemu-debian-13-amd64
mv debian-13-nocloud-amd64.qcow2 qemu-debian-13-amd64/
```

## 2. Prepare the image and startup script

Verify and adjust the qcow2 disk size:
```bash
# Check qcow2 image virtual size (default only 3 GB)
qemu-img info debian-13-nocloud-amd64.qcow2

# Resize qcow2 image virtual size to 20 GB
qemu-img resize debian-13-nocloud-amd64.qcow2 20G
```

Create a start shell for qcow2 image.

```bash
tee start.sh > /dev/null <<'EOF'
#!/bin/sh

qemu-system-x86_64 \
    -m 2G \
    -smp 2 \
    -drive file=debian-13-nocloud-amd64.qcow2,format=qcow2,if=virtio \
    -netdev user,id=net0,hostfwd=tcp::2222-:22 \
    -device virtio-net-pci,netdev=net0 \
    -enable-kvm \
    -nographic
EOF

chmod + x start.sh
```

Start the qemu machine with the shell, using `sudo` in order to use KVM:
```bash
sudo ./start.sh
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
fd0       2:0    1    4K  0 disk
sr0      11:0    1 1024M  0 rom
vda     254:0    0   20G  0 disk
├─vda1  254:1    0 19.9G  0 part /
├─vda14 254:14   0    3M  0 part
└─vda15 254:15   0  124M  0 part /boot/efi
```

Note: qmeu machine can connected via port 2222 after openssh-server is configured.

(END)