# UART Logs

> **历史诊断资料**：这些内容记录设备恢复前的排查过程。设备已经成功恢复；当前实机验证恢复流程请以仓库根目录 `README.md` 为准。


This directory stores raw Xiaomi Router 3G serial console captures.

Required first captures:

    R3G-normal-boot.txt
    R3G-reset-boot.txt

Serial configuration:

    3.3V TTL
    115200 baud
    8 data bits
    1 stop bit
    no parity
    no flow control

First-pass wiring:

    R3G GND -> USB-TTL GND
    R3G TX  -> USB-TTL RX

Do not connect VCC.

For the initial diagnostic capture, router RX can remain disconnected so the session stays read-only.

Raw UART logs should remain unedited.
