# Xiaomi Router 3G Recovery / Reverse Engineering

Recovery, boot diagnostics, UART analysis, firmware research, and reverse-engineering notes for the Xiaomi Router 3G (R3G).

Current project phase:

    read-only UART diagnosis before any flash write

## Device

    Model: Xiaomi Router 3G
    Platform: MediaTek MT7621
    Previous firmware environment: OpenWrt
    Previous bootloader environment: Breed

The router became abnormal after flashing an unidentified firmware image.

## Current hardware symptom

Normal power-on:

    Power on
    Orange LED turns on
    Orange LED remains continuously on

Hold RESET while powering on:

    Hold RESET
    Power on
    Orange LED turns on
    LED goes off briefly
    Orange LED turns on again
    Orange LED remains on

RESET changes the early boot behavior.

This does not by itself prove whether Breed is still intact or has been overwritten.

## Network recovery tests

Tested:

    192.168.1.1    NO RESPONSE
    192.168.31.1   NO RESPONSE

Breed Web recovery is currently not reachable.

## Current recovery strategy

Next diagnostic step:

    UART / TTL read-only boot log capture

Required logs:

    uart/R3G-normal-boot.txt
    uart/R3G-reset-boot.txt

The logs will be compared to determine where boot stops:

    BootROM
    Bootloader / Breed
    Flash / partition initialization
    Kernel
    Root filesystem / OpenWrt

## UART baseline

Use 3.3V TTL only.

    Baud:         115200
    Data bits:    8
    Stop bits:    1
    Parity:       None
    Flow control: None

Safest first connection:

    R3G GND -> USB-TTL GND
    R3G TX  -> USB-TTL RX

Initial read-only capture:

    R3G RX  -> NOT CONNECTED
    VCC     -> DO NOT CONNECT

## Safety policy

Until UART logs and flash layout are understood, do not:

    erase NAND
    write NAND
    overwrite bootloader
    flash arbitrary full dumps
    overwrite Factory
    overwrite EEPROM
    overwrite Bdata
    overwrite Config

Factory / EEPROM may contain device-specific data such as:

    Wi-Fi calibration
    MAC addresses
    board-specific parameters

## Repository structure

    Xiaomi-Router-3G/
    ├── README.md
    ├── SHA256SUMS
    ├── docs/
    │   ├── README.md
    │   ├── Xiaomi_R3G_RECOVERY_DIAGNOSIS.md
    │   ├── Xiaomi_R3G_TTL_PUTTY_GUIDE.md
    │   ├── Xiaomi_R3G_BREED_NETWORK_RECOVERY_TEST.md
    │   └── Xiaomi_R3G_MASTER_INDEX.md
    ├── uart/
    │   └── README.md
    ├── firmware/
    │   └── README.md
    ├── dumps/
    │   └── README.md
    └── notes/
        └── README.md

## Current project state

    BREED_WEB=NOT_REACHABLE
    IP_192_168_1_1=NO_RESPONSE
    IP_192_168_31_1=NO_RESPONSE
    RESET_BEHAVIOR_DIFFERENT=YES
    UART_CAPTURE=NEXT
    FLASH_WRITE=FORBIDDEN_FOR_NOW

## Next milestone

Capture:

    uart/R3G-normal-boot.txt
    uart/R3G-reset-boot.txt

Then compare both boot paths before any recovery write operation.
