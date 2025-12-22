[English](README.md) | [简体中文](README.zh-CN.md)

# OpenWrt for Xiaomi AX3000
> **Supported device**: Xiaomi Router AX3000

---

## Hard requirements

1. **You must flash the stock `rootfs`, not `rootfs_1`.**  
   Reason: the partition table has been expanded. I merged the stock `rootfs_1` partition and the following `overlay` partition into one large `overlay`. If you flash `rootfs_1`, the partition layout will not match and you may brick the device.  
   You can check which slot you are currently booted from with `cat /proc/cmdline`. The detailed flashing steps are below.

---

## Prerequisites

- **UART is strongly recommended, and make sure you can enter U-Boot**: any environment variable / UBI operations should be done with a controllable serial console.
- **Download Xiaomi's official recovery/unbrick tool yourself.**

---

## Flashing guide

- **Get a stock shell**: you can use [xmir-patcher](https://github.com/openwrt-xiaomi/xmir-patcher).
- **Switching slots**:  
  You must flash to **mtd18**, i.e. `rootfs`. Therefore, the system you are currently running **must not** be on `rootfs`, otherwise you cannot write to the active system partition.  
  Check your current slot with `cat /proc/cmdline`. If you see something like:

  ```text
  ubi.mtd=rootfs_1 root=mtd:ubi_rootfs rootfstype=squashfs cnss2.bdf_integrated=0x24 cnss2.bdf_pci0=0x60 cnss2.bdf_pci1=0x60 cnss2.skip_radio_bmap=4 rootwait uart_en=1 swiotlb=1
  ```

  then you are running on `rootfs_1`.

  If you are currently on `rootfs`, there are two ways to switch:

  1) **Recommended**: re-flash once using Xiaomi's official recovery tool. It usually toggles the boot slot automatically: if you were on `rootfs` before flashing, you will boot from `rootfs_1` after flashing.

  2) **Faster (not recommended for long-term use)**: clone everything from `mtd18` (`rootfs`) to `mtd19` (`rootfs_1`), then reboot. Example:

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

  After this, you should boot from `rootfs_1`. This is fine as a temporary step for flashing, but not recommended as a permanent setup.

- **Flashing**:  
  Upload the factory UBI via `scp`, for example:

  ```bash
  scp -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedAlgorithms=+ssh-rsa -O openwrt-qualcommax-ipq50xx-xiaomi_ax3000-squashfs-factory.ubi root@192.168.31.1:/tmp/
  ```

  Then, while running from `rootfs_1`, flash `mtd18`:

  ```bash
  ubiformat /dev/mtd18 -f openwrt-qualcommax-ipq50xx-xiaomi_ax3000-squashfs-factory.ubi
  nvram set flag_try_sys1_failed=0
  nvram set flag_boot_rootfs=0      # pin slot 0 / rootfs
  nvram set flag_try_sys2_failed=8  # make slot 1 / rootfs_1 never be tried
  nvram set flag_last_success=0
  nvram set flag_boot_success=1
  nvram commit
  reboot
  ```

  After flashing, the IP address is `192.168.31.1` (same as stock Xiaomi firmware).

---

## Known issues

- **5 GHz Wi‑Fi does not work properly**: the 5 GHz radio cannot transmit (no beacon), but it can scan and detect nearby APs. The preliminary suspicion is a driver compatibility issue between `ath11k` and `QCN6122`. Track upstream fixes (ath11k/mac80211/firmware) and follow up once upstream is fixed.
- With only **256 MB RAM**, enabling both radios can easily cause OOM. I have bundled `zram-swap`; it is recommended to disable one radio anyway (since 5 GHz currently cannot be used).

---

## Screenshots

<img width="1260" height="907" alt="iShot 2025-12-20 23 06 43" src="https://github.com/user-attachments/assets/6a07db5e-5f75-41e1-ab6d-1eaa85427aaf" />
<img width="1215" height="532" alt="image" src="https://github.com/user-attachments/assets/701f407b-d907-4c2a-9a76-d00a7f9b3b91" />
