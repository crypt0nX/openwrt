[English](README_EN.md) | [简体中文](README.zh-CN.md)

# OpenWrt for Xiaomi AX3000
> **适用设备**：小米路由器 AX3000

---

## 硬性要求

1. **必须刷入原厂的 `rootfs`，不能是 `rootfs_1`**  
   原因：分区表已扩容，原厂的`rootfs_1`以及后面的`overlay`分区被我合并成了一个大的`overlay`；刷入 `rootfs_1` 会造成分区表不匹配，可以用 `cat /proc/cmdline`，看自己现在在哪个。怎么刷具体做法在下面。

---

## 前置准备

- **建议准备 UART 并确认可进入 U-Boot**：任何环境变量/UBI 操作都建议在可控的串口环境下进行。
- **自行下载原厂救砖工具**。

---

## 刷机方法

- **进入原厂shell**：可以用[xmir-patcher](https://github.com/openwrt-xiaomi/xmir-patcher)。
- **切换方法**：
因为一定要刷到mtd18也就是`rootfs`。所以你当前刷机的系统不能在`rootfs`，不然没法在系统盘上刷，先用`cat /proc/cmdline`看自己在哪个，如果是
`ubi.mtd=rootfs_1 root=mtd:ubi_rootfs rootfstype=squashfs cnss2.bdf_integrated=0x24 cnss2.bdf_pci0=0x60 cnss2.bdf_pci1=0x60 cnss2.skip_radio_bmap=4 rootwait uart_en=1 swiotlb=1`
就说明自己是在`rootfs_1`。
但如果你现在是在`rootfs`，提供两种切换的方法。第一种比较正规，就是用原厂的救砖工具重刷一次，默认会切换一次系统分区，如果刷机前在`rootfs`，刷完后的系统就在`rootfs_1`。第二种就比较快。适合嫌麻烦的，具体做法是直接在`rootfs`上把`mtd18`的东西全拷贝到`mtd19`（也就是`rootfs_1`）,参考以下命令：
```bash
cd /tmp
umount /dev/mtdblock19 2>/dev/null
dd if=/dev/mtdblock18 of=/dev/mtdblock19 bs=1M conv=fsync

nvram set flag_try_sys2_failed=0
nvram set flag_boot_rootfs=1
nvram set flag_last_success=1
nvram commit
reboot
```
这样弄完后你应该会从`rootfs_1`启动，不推荐作为长期使用方法，因为你克隆了一个状态是连续的区域，但是作为刷机临时的够了。

- **刷机方法**：
固件可以用`scp`传上去：`scp  -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedAlgorithms=+ssh-rsa -O penwrt-qualcommax-ipq50xx-xiaomi_ax3000-squashfs-factory.ubi  root@192.168.31.1:/tmp/`
然后在`rootfs_1`：
```bash
ubiformat /dev/mtd18 -f openwrt-qualcommax-ipq50xx-xiaomi_ax3000-squashfs-factory.ubi
nvram set flag_try_sys1_failed=0
nvram set flag_boot_rootfs=0      # 固定槽0/rootfs
nvram set flag_try_sys2_failed=8  # 让槽1/rootfs_1 永远别尝试
nvram set flag_last_success=0
nvram set flag_boot_success=1
nvram commit
reboot
```
刷完后ip地址是`192.168.31.1`，和小米自带的一样。

## 已知问题

- **5G WIFI无法正常工作**：5G 暂时无法发射信号，但可扫描到周边热点。初步判断是疑似`ath11k`对`QCN6122`的驱动适配问题。需要关注上游（ath11k/mac80211/firmware）修复进展，待上游修复后再跟跟进。
- 因为内存只有256M所以两个WIFI一起开容易造成OOM，我已经内置了`zarm-swap`，建议关闭一个（反正5g也用不了）。

---

## 截图

<img width="1260" height="907" alt="iShot 2025-12-20 23 06 43" src="https://github.com/user-attachments/assets/6a07db5e-5f75-41e1-ab6d-1eaa85427aaf" />
<img width="1215" height="532" alt="image" src="https://github.com/user-attachments/assets/701f407b-d907-4c2a-9a76-d00a7f9b3b91" />


