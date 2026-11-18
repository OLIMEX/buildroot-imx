Olimex iMX8MP-SOM-4GB-IND + iMX8MP-SOM-EVB
==========================================

Ready image and boot loader (built from this tree):
  https://images.olimex.com/release/imx8mp/buildroot/
  buildroot-20261118-ddr3600_odt60_dbi.img.gz             complete image
  imx8mp-olimex-buildroot-u-boot-lpddr4-3600-dbi.bin      boot loader only

Build
-----

  make olimex_imx8mpevk_defconfig
  make

The result is in output/images/:
  sdcard.img          - complete microSD card image
  imx8-boot-sd.bin    - boot loader only (SPL, DDR firmware, ATF, U-Boot)

Write the complete image (this erases the card; replace sdX):

  sudo dd if=output/images/sdcard.img of=/dev/sdX bs=4M conv=fsync status=progress

LPDDR4 settings (U-Boot SPL)
----------------------------

U-Boot patch 0002 sets up the Micron MT53D1024M32D4DT-053 LPDDR4 with
3600 MT/s, DRAM DQ ODT 60 ohm and read/write DBI, and the defconfig selects
the 2020.06 DDR PHY training firmware (BR2_PACKAGE_FIRMWARE_IMX_DDR_VERSION).
Older images used the NXP EVK settings (4000 MT/s, DQ ODT 40 ohm, old
training firmware), which give rare memory read errors under load.
4000 MT/s is also above the rating of the MT53D1024M32D4DT-053 (speed
grade -053: at most 3733 MT/s); 3600 MT/s is within it.

The serial console (115200 8N1) shows at power on:

  DDRINFO: DRAM rate 3600MTS

Update only the boot loader on an existing card
-----------------------------------------------

This keeps the system and the data. The boot loader is written 32 KiB from
the start of the card; the partitions are not touched. Double check the
device name - dd does not ask.

On a PC, with the card in a card reader:

  sudo dd if=output/images/imx8-boot-sd.bin of=/dev/sdX bs=1k seek=32 conv=fsync

Or on the board itself, booted from the microSD card (/dev/mmcblk1):

  dd if=imx8-boot-sd.bin of=/dev/mmcblk1 bs=1k seek=32 conv=fsync
  sync; reboot

Use the Buildroot boot loader only with Buildroot images. The Armbian
(Debian) images use a different U-Boot environment, see
https://github.com/OLIMEX/imx8mp-build/tree/bookworm/doc/uboot-ddr
