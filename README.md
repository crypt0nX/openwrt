# OpenWrt for Xiaomi AX3000
> **适用设备**：小米路由器 AX3000

---

## 硬性要求

1. **必须刷入原厂的 `rootfs`，不能是 `rootfs_1`**  
   原因：分区表已扩容，启动与挂载链路依赖当前布局；刷入 `rootfs_1` 存在与现有布局不匹配的高风险，可以用 `cat /proc/cmdline`，看自己现在在哪个。

2. **可选：先用原厂救砖工具刷回一次**  
   目的：此时原厂逻辑会自动切换到 `rootfs_1`。

---

## 前置准备

- **建议准备 UART 并确认可进入 U-Boot**：任何环境变量/UBI 操作都建议在可控的串口环境下进行。
- **自行下载原厂救砖工具**。

---

## 刷机方法

- **进入原厂shell**：可以用[xmir-patcher](https://github.com/openwrt-xiaomi/xmir-patcher)。
- **刷机方法（示例）**：一定要刷到mtd18也就是rootfs。
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


