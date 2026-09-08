# Xiaomi Router 3G（R3G）TTL + PuTTY 指南

> **历史诊断资料**：这些内容记录设备恢复前的排查过程。设备已经成功恢复；当前实机验证恢复流程请以仓库根目录 `README.md` 为准。


## USB-TTL
必须使用 3.3V TTL。常见：CH340、CP2102、FT232。

## 第一轮最安全接线
R3G GND -> USB-TTL GND
R3G TX  -> USB-TTL RX

暂时不接 R3G RX。
VCC 绝对不要接。

## PuTTY
Connection type: Serial
Serial line: 实际 COM 号
Speed: 115200

Serial 设置：
- Data bits: 8
- Stop bits: 1
- Parity: None
- Flow control: None

即 115200 8N1，无流控。

## 日志
Session -> Logging -> All session output

普通启动：
R3G-normal-boot.txt

RESET 启动：
R3G-reset-boot.txt

## 普通启动
先开 PuTTY，再给路由器正常上电。不要按键，完整保存日志。

## RESET 启动
断电，按住 RESET，上电，观察橙灯“亮 → 灭一下 → 再亮”，再松开 RESET，完整保存日志。

## 黑屏优先检查
COM 号、共地、TX/RX 是否接反、是否 3.3V、115200、焊点是否正确。
