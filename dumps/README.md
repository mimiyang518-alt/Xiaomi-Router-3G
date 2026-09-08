# Flash Dumps

This directory is reserved for flash backup metadata and verified dumps.

Preferred order before any destructive recovery:

    1. identify flash device
    2. identify partition layout
    3. read existing partitions
    4. calculate hashes
    5. preserve device-specific regions
    6. only then evaluate repair writes

Treat these as protected until proven otherwise:

    Factory
    EEPROM
    Bdata
    Config
    calibration data
    MAC-address data

Recommended naming:

    R3G_FACTORY_READONLY_YYYYMMDD.bin
    R3G_BOOTLOADER_READONLY_YYYYMMDD.bin
    R3G_PARTITION_TABLE_YYYYMMDD.txt

Always record SHA256.
