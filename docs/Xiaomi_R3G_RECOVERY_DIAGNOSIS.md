# Xiaomi Router 3G（R3G）启动异常与恢复诊断记录

## 设备背景
- Xiaomi Router 3G（R3G）
- MediaTek MT7621
- 曾刷过 OpenWrt、Breed
- 在刷某个固件过程中开始出现异常

## 已确认现象
普通上电：橙灯一直亮；192.168.1.1、192.168.31.1 均无响应。
按住 RESET 上电：橙灯亮 → 灭一下 → 再橙灯常亮；Breed Web 无法进入；两个常见地址均无响应。

## 当前判断
RESET 启动与普通启动不同，说明 RESET 在启动早期至少被某一级代码检测到。仅凭 LED 不能证明 Breed 已彻底损坏。

可能原因：
- Breed 被覆盖或部分损坏
- Bootloader 仍在，但恢复网络初始化失败
- Kernel / 固件损坏
- NAND / UBI / 分区布局异常
- 启动参数异常

## 当前策略
停止猜恢复 IP，转入 TTL/UART 只读诊断。

## 安全原则
暂时不要随机刷固件、full dump、擦 NAND、覆盖 Bootloader、Factory、EEPROM、Bdata、Config。

下一步抓：
- R3G-normal-boot.txt
- R3G-reset-boot.txt
