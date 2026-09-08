# Xiaomi Router 3G (R3G) 实机验证救砖指南

> **R3G 橙灯常亮，RESET 也无法恢复，但 TTL 有输出？从这里开始。**

这份流程来自一次实际成功恢复 Xiaomi Router 3G 的完整过程。目标是尽量少做危险操作：**不手工执行 NAND erase/write，不覆盖 Factory/EEPROM/校准数据，不刷整片 NAND dump。**

最终成功结果：原厂 MiWiFi R3G 2.25.124 正常启动，LAN 恢复为 `192.168.31.1`，无线恢复，串口出现 `Booting up finished.`，设备亮蓝灯。

---

## 适用场景

- Xiaomi Router 3G / R3G
- 正常开机橙灯常亮，无法进入系统
- RESET 常规恢复也失败
- Breed Web 无法访问
- 但 3.3V TTL 串口仍然有 U-Boot 输出，并且可以进入启动菜单

如果 TTL 完全没有输出，这不是本文覆盖的恢复路径。

---

## 准备

### 1. TTL

- **只能使用 3.3V TTL，绝对不要使用 5V**
- GND ↔ GND
- R3G TX → TTL RX
- R3G RX → TTL TX
- **不要连接 VCC**
- 串口参数：`115200 8N1`
- Flow Control：None

### 2. 电脑网络

实机成功时：

```text
PC / TFTP Server : 192.168.31.100
R3G              : 192.168.31.1
Subnet Mask      : 255.255.255.0
Gateway          : 留空
DNS              : 留空
```

如果 TFTP 一直出现 `T T T T ...`，先检查：

- 网线是否正确连接
- TFTP Server 是否绑定到了正确网卡
- Windows 防火墙是否拦截
- 文件是否位于 TFTP 根目录
- 网卡 Gateway 是否留空

### 3. 文件

需要：

```text
r3g-kernel.bin
miwifi_r3g_firmware_12f97_2.25.124-dev.bin
```

后者在 USB Recovery 阶段改名为：

```text
firmware.bin
```

---

# 第一阶段：菜单 4，先把正确 kernel 送入 RAM 并验证

开机时进入 U-Boot 菜单。

菜单中应看到类似：

```text
1: Load system code to SDRAM via TFTP.
2: Load system code then write to Flash via TFTP.
3: Boot system code via Flash (default).
4: Entr boot command line interface.
7: Load Boot Loader code then write to Flash via Serial.
9: Load Boot Loader code then write to Flash via TFTP.
```

**第一次选择：**

```text
4
```

进入命令行后应看到：

```text
MT7621 #
```

执行：

```text
tftpboot 83000000 r3g-kernel.bin
```

成功时必须看到：

```text
TFTP from server 192.168.31.100; our IP address is 192.168.31.1
Filename 'r3g-kernel.bin'.
Load address: 0x83000000
...
done
Bytes transferred = 3498340 (356164 hex)
LoadAddr=83000000 NetBootFileXferSize= 00356164
```

如果传输大小不是 `3498340`，**停止，不要继续写 Flash。**

接着验证 RAM 中的镜像：

```text
md.b 83000000 40
```

正确结果开头应类似：

```text
83000000: 27 05 19 56 ...
83000020: 4d 49 50 53 20 4f 70 65 6e 57 72 74 ...
83000030: 75 78 2d 33 2e 31 30 2e 31 34 ...
```

也就是能看到：

```text
MIPS OpenWrt Linux-3.10.14
```

如果不是这个头，**停止。**

确认 kernel 正确后执行：

```text
setenv bootfile r3g-kernel.bin
saveenv
reset
```

路由器会重启。

---

# 第二阶段：重启后选择菜单 2，自动恢复双 kernel

再次进入 U-Boot 菜单。

这次选择：

```text
2
```

菜单 2 的含义是：

```text
Load system code then write to Flash via TFTP.
```

出现：

```text
Warning!! Erase Linux in Flash then burn new one. Are you sure?(Y/N)
```

输入：

```text
Y
```

接下来确认参数：

```text
Input device IP (192.168.31.1) ==:
```

直接 Enter。

```text
Input server IP (192.168.31.100) ==:
```

直接 Enter。

文件名必须是：

```text
r3g-kernel.bin
```

如果默认不是这个文件名，输入 `r3g-kernel.bin` 后 Enter。

然后**不要再按任何键，不要断电。**

成功过程中会看到类似：

```text
Bytes transferred = 3498340
Writing image to 0x200000
...
Done!
Writing image to 0x600000
...
Done!
```

这一步由 U-Boot 自动恢复两个 kernel 区。

**不要自己执行 `nand erase`、`nand write`、`cp` 等命令。**

---

# 第三阶段：让原厂 kernel 启动并进入 USB Recovery

双 kernel 修复后，原厂 kernel 应能正常校验、解压并启动。成功日志中出现：

```text
Booting image at bc200000 ...
Image Name: MIPS OpenWrt Linux-3.10.14
Data Size: 3498273 Bytes = 3.3 MB
Load Address: 80001000
Entry Point: 8046e1a0
Verifying Checksum ... OK
Uncompressing Kernel Image ... OK
...
Starting kernel ...
LINUX started...
```

并可识别为：

```text
MiWiFi-R3G-2.25.124
```

如果 rootfs0/rootfs1 都损坏，系统会进入：

```text
Check for USB recovery...
Both systems are corrupted... Entering recovery mode
Press reset button to enter USB recovery
```

到这里，kernel 已经修复，接下来交给小米原厂 USB Recovery。

---

# 第四阶段：FAT32 U 盘恢复完整原厂系统

准备一个 **FAT32** U 盘。

将完整原厂 DEV 固件：

```text
miwifi_r3g_firmware_12f97_2.25.124-dev.bin
```

改名为：

```text
firmware.bin
```

U 盘根目录只放这个恢复文件最稳妥。

插入 R3G USB 口。

确认串口仍停在：

```text
Press reset button to enter USB recovery
```

然后：

1. 按住 RESET 大约 3 秒。
2. 看到 LED 状态变化后松开。
3. **之后不要断电、不要拔 U 盘、不要再次按 RESET。**

Recovery 会自动处理完整固件。

实机日志确认它会执行：

```text
System recovery Burning uImage.bin to kernel0 ...
Writing from uImage.bin to kernel0 ...
Done

Burning uImage.bin to kernel1 ...
Writing from uImage.bin to kernel1 ...
Done

Burning root.ubi to rootfs0 ...
ubiformat: mtd10 (nand), size 33554432 bytes (32.0 MiB)
...

Burning root.ubi to rootfs1 ...
ubiformat: mtd11 (nand), size 33554432 bytes (32.0 MiB)
...
```

也就是说原厂 USB Recovery 会自动恢复：

```text
kernel0
kernel1
rootfs0
rootfs1
```

并重新初始化 overlay。

---

# 成功判定

最终实机成功日志出现：

```text
system type(R3G/2): SQUASH/3
ROOTFS: /dev/mtdblock13 on / type squashfs
boot_check[8507]: Booting up finished.
```

同时：

- MiWiFi R3G 2.25.124 正常启动
- LAN 恢复为 `192.168.31.1`
- 2.4G / 5G 无线恢复
- LED 最终变为蓝色

浏览器访问：

```text
http://192.168.31.1
```

确认系统正常后即可拔掉 U 盘。

---

# 最重要的安全规则

1. **TTL 必须是 3.3V，绝对不要接 5V。**
2. **不要连接 TTL VCC。**
3. 第一阶段 `r3g-kernel.bin` 的 TFTP 大小不是 `3498340` 时不要继续。
4. `md.b 83000000 40` 看不到正确 uImage 头时不要继续。
5. 菜单 2 写 Flash 后不要断电。
6. USB Recovery 开始写 NAND 后不要拔 U 盘、不要按 RESET。
7. **不要手工执行 NAND erase/write 来恢复 rootfs。**
8. **不要刷整片 NAND dump。**
9. **不要覆盖 Factory / EEPROM / MAC / Wi-Fi calibration 数据。**

---

# 一页版流程

```text
TTL 115200 8N1
        ↓
第一次启动菜单选 4
        ↓
tftpboot 83000000 r3g-kernel.bin
        ↓
确认：3498340 bytes
        ↓
md.b 83000000 40
确认：27 05 19 56 / MIPS OpenWrt Linux-3.10.14
        ↓
setenv bootfile r3g-kernel.bin
saveenv
reset
        ↓
第二次启动菜单选 2
        ↓
Y
        ↓
Device IP = 192.168.31.1
Server IP = 192.168.31.100
File      = r3g-kernel.bin
        ↓
U-Boot 自动恢复 kernel0 + kernel1
        ↓
原厂 kernel 启动
        ↓
Press reset button to enter USB recovery
        ↓
FAT32 USB
firmware.bin
(来自 miwifi_r3g_firmware_12f97_2.25.124-dev.bin)
        ↓
RESET 按住约 3 秒
        ↓
原厂 USB Recovery
        ↓
kernel0 + kernel1 + rootfs0 + rootfs1 + overlay
        ↓
自动重启
        ↓
Booting up finished.
        ↓
蓝灯
        ↓
恢复成功
```

---

## 说明

本文只保留实际成功恢复链路。HDR1 / Bad Magic Number、错误 NAND 写入实验、固件结构分析、分区研究和历史调试过程应单独放到 `notes/` 或 `docs/research/`，不要混入主救砖流程。
