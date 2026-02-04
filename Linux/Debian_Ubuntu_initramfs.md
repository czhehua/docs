# Debian/Ubuntu initfamfs 加载模块优化

虚拟机/容器中的 Debian/Ubuntu 通过设置 initfamfs 加载的模块，让其保持精简，加速启动。

方法：

```
sudo nano /etc/initramfs-tools/initramfs.conf
# 将 MODULES=most 改为 MODULES=dep
sudo update-initramfs -u
```

效果：
- /boot/initrd.img-$(uname -r) 大小下降（从 50MB+ → 20MB-）
- 系统启动更快
- 不影响 VM 网络和磁盘

物理机由于需要加载各种硬件驱动，建议保持 `MODULES=most` 原样。

参考：
- [深入理解 initfamfs](https://geek-blogs.com/blog/initramfs-linux/)