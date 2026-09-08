# Xiaomi Router 3G Recovery / Reverse Engineering

Repository:
mimiyang518-alt/Xiaomi-Router-3G

Device:
- Xiaomi Router 3G
- MediaTek MT7621

Current symptom:
- Normal power-on: orange LED remains on
- Hold RESET during power-on: orange LED on -> briefly off -> orange on
- Breed recovery unreachable
- 192.168.1.1 no response
- 192.168.31.1 no response

Recovery policy:
- Do not write NAND yet
- Do not overwrite Factory / EEPROM
- Do not flash full dumps blindly
- First capture UART logs

UART:
- 3.3V TTL only
- 115200 8N1
- No flow control
- Do NOT connect VCC

Suggested directories:
uart/
firmware/
dumps/
notes/

Recommended logs:
uart/R3G-normal-boot.txt
uart/R3G-reset-boot.txt
