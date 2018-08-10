# Installing OpenBSD/arm64 on RockPro64

This short guide might be useful when installing OpenBSD onto the new RockPro64 boards by PINE64.
It's perhaps a bit rough still (e.g. the dtb file isn't in the `dtb` package at this moment), but
it does get the job done.

The steps I took were basically:
- flash u-boot into the onboard SPI memory
- obtain the device tree file for this particular board
- write the miniroot and include the dtb file

## Requirements

- [RockPro64](https://www.pine64.org/?page_id=61454)
- USB-serial adapter capable of 1500000 baud rate, e.g. based on CP2104 or CH341 chips
- uSD card

## Flashing u-boot

Start by flashing the boatloader to the SPI flash. Grab a `u-boot-flash-spi-rockpro64.img.xz` from [ayufan-rock64/linux-u-boot](https://github.com/ayufan-rock64/linux-u-boot/releases), extract and write to an uSD card:

```
xz -d u-boot-flash-spi-rockpro64.img.xz
dd if=u-boot-flash-spi-rockpro64.img of=/dev/rsdXc bs=1m
```
Hook up the console to the board and wait for it to finish flashing.

## Preparing the installation media

Download the dtb file from this repository, which in turn was extracted from [linux-image-4.4.138-1094-rockchip-ayufan-gf13a8a9a4eee\_4.4.138-1094-rockchip-ayufan\_arm64.deb](https://github.com/ayufan-rock64/linux-kernel/releases/download/4.4.138-1094-rockchip-ayufan/linux-image-4.4.138-1094-rockchip-ayufan-gf13a8a9a4eee_4.4.138-1094-rockchip-ayufan_arm64.deb)

```
dd if=miniroot63.fs of=/dev/rsdXc bs=1m
mount /dev/sdXi /mnt
mkdir -p /mnt/dtb/rockchip/
cp rk3399-rockpro64.dtb /mnt/dtb/rockchip/
umount /mnt
```

Pop the card into the board and boot:

```
U-Boot SPL board init

U-Boot SPL 2017.09-rockchip-ayufan-1033-gdf02018479 (Aug 06 2018 - 22:29:15)
booted from SPI flash
Trying to boot from SPI
NOTICE:  BL31: v1.3(debug):65aa5ce
NOTICE:  BL31: Built : 10:47:37, Jun 19 2018
NOTICE:  BL31: Rockchip release version: v1.1
INFO:    GICv3 with legacy support detected. ARM GICV3 driver initialized in EL3
INFO:    Using opteed sec cpu_context!
INFO:    boot cpu mask: 0
INFO:    plat_rockchip_pmu_init(1151): pd status 3e
INFO:    BL31: Initializing runtime services
WARNING: No OPTEE provideSMC_UNK
ERROR:   Error initializing runtime service opteed_fast
INFO:    BL31: Preparing for EL3 exit to normal world
INFO:    Entry point address = 0x200000
INFO:    SPSR = 0x3c9


U-Boot 2017.09-rockchip-ayufan-1033-gdf02018479 (Aug 06 2018 - 22:29:23 +0000)

Model: Pine64 RockPro64
DRAM:  3.9 GiB
DCDC_REG1@vdd_center: ; enabling
DCDC_REG2@vdd_cpu_l: ; enabling
DCDC_REG3@vcc_ddr: ; enabling (ret: -38)
DCDC_REG4@vcc_1v8: set 1800000 uV; enabling
LDO_REG1@vcc1v8_dvp: set 1800000 uV; enabling
LDO_REG2@vcc3v0_touch: set 3000000 uV; enabling
LDO_REG3@vcc1v8_pmu: set 1800000 uV; enabling
LDO_REG4@vcc_sd: set 3300000 uV; enabling
LDO_REG5@vcca3v0_codec: set 3000000 uV; enabling
LDO_REG6@vcc_1v5: set 1500000 uV; enabling
LDO_REG7@vcca1v8_codec: set 1800000 uV; enabling
LDO_REG8@vcc_3v0: set 3000000 uV; enabling
SWITCH_REG1@vcc3v3_s3: ; enabling (ret: -38)
SWITCH_REG2@vcc3v3_s0: ; enabling (ret: -38)
vcc1v8-s0@vcc1v8_s0: set 1800000 uV; enabling (ret: -38)
dc-12v@dc_12v: set 12000000 uV; enabling (ret: -38)
vcc-sys@vcc_sys: set 5000000 uV; enabling (ret: -38)
vcc3v3-sys@vcc3v3_sys: set 3300000 uV; enabling (ret: -38)
vcc-phy-regulator@vcc_phy: ; enabling (ret: -38)
vdd-log@vdd_log: ; enabling (ret: -38)
MMC:   sdhci@fe330000: 0, dwmmc@fe320000: 1
SF: Detected gd25q128 with page size 256 Bytes, erase size 4 KiB, total 16 MiB
*** Warning - bad CRC, using default environment

In:    serial@ff1a0000
Out:   serial@ff1a0000
Err:   serial@ff1a0000
Model: Pine64 RockPro64
Net:   eth0: ethernet@fe300000
Hit any key to stop autoboot:  0 
Card did not respond to voltage select!
mmc_init: -95, time 19
switch to partitions #0, OK
mmc1 is current device
Scanning mmc 1:1...
reading /dtb/rockchip/rk3399-rockpro64.dtb
67805 bytes read in 12 ms (5.4 MiB/s)
Found EFI removable media binary efi/boot/bootaa64.efi
reading efi/boot/bootaa64.efi
108780 bytes read in 16 ms (6.5 MiB/s)
## Starting EFI application at 02000000 ...
Card did not respond to voltage select!
mmc_init: -95, time 20
Scanning disk sdhci@fe330000.blk...
Scanning disk dwmmc@fe320000.blk...
Found 2 disks
>> OpenBSD/arm64 BOOTAA64 0.13
boot> 
```

You'll want to ensure it says `reading /dtb/rockchip/rk3399-rockpro64.dtb`.

Proceed with the installation as usual, but be aware that after the installation you'll have to
put the dtb file on the `i` partition again otherwise your first boot may end up hanging after:

```
root on sd0a (f3e4119d408a5caa.a) swap on sd0b dump on sd0b
```

So after you've finished installing, eject the SD card and write the dtb to it again from
another machine:

```
mount /dev/sdXi /mnt
mkdir -p /mnt/dtb/rockchip/
cp rk3399-rockpro64.dtb /mnt/dtb/rockchip/
umount /mnt
```

Now the board can be booted as usual:

```
>> OpenBSD/arm64 BOOTAA64 0.13
boot>
booting sd0a:/bsd: 4718036+735424+644656+938928 [302281+96+607992+325471]=0xab08a8
[...]
[ using 1236680 bytes of bsd ELF symbol table ]
Copyright (c) 1982, 1986, 1989, 1991, 1993
        The Regents of the University of California.  All rights reserved.
Copyright (c) 1995-2018 OpenBSD. All rights reserved.  https://www.OpenBSD.org

OpenBSD 6.3-current (GENERIC.MP) #103: Thu Aug  9 06:44:15 MDT 2018
    deraadt@arm64.openbsd.org:/usr/src/sys/arch/arm64/compile/GENERIC.MP
real mem  = 4098547712 (3908MB)
avail mem = 3939385344 (3756MB)
mainbus0 at root: Pine64 RockPro64
cpu0 at mainbus0 mpidr 0: ARM Cortex-A53 r0p4
efi0 at mainbus0: UEFI 2.0.5
efi0: Das U-boot rev 0x0
psci0 at mainbus0: PSCI 1.0
agintc0 at mainbus0 nirq 288, nredist 6 ipi: 0, 1: "interrupt-controller"
agintcmsi0 at agintc0
syscon0 at mainbus0: "qos"
syscon1 at mainbus0: "qos"
syscon2 at mainbus0: "qos"
syscon3 at mainbus0: "qos"
syscon4 at mainbus0: "qos"
syscon5 at mainbus0: "qos"
syscon6 at mainbus0: "qos"
syscon7 at mainbus0: "qos"
syscon8 at mainbus0: "qos"
syscon9 at mainbus0: "qos"
syscon10 at mainbus0: "qos"
syscon11 at mainbus0: "qos"
syscon12 at mainbus0: "qos"
syscon13 at mainbus0: "qos"
syscon14 at mainbus0: "qos"
syscon15 at mainbus0: "qos"
syscon16 at mainbus0: "qos"
syscon17 at mainbus0: "qos"
syscon18 at mainbus0: "qos"
syscon19 at mainbus0: "qos"
syscon20 at mainbus0: "qos"
syscon21 at mainbus0: "qos"
syscon22 at mainbus0: "qos"
syscon23 at mainbus0: "qos"
syscon24 at mainbus0: "qos"
syscon25 at mainbus0: "power-management"
syscon26 at mainbus0: "syscon"
rkclock0 at mainbus0
rk3399_pmu_set_frequency: 0x0000002c
rkclock1 at mainbus0
rk3399_set_frequency: 0x000000d9
rk3399_set_frequency: 0x000001d9
rk3399_set_frequency: 0x000000db
rk3399_set_frequency: 0x000001db
rk3399_set_frequency: 0x000000d0
syscon27 at mainbus0: "syscon"
rkpinctrl0 at mainbus0: "pinctrl"
rkgpio0 at rkpinctrl0
rkgpio1 at rkpinctrl0
rkgpio2 at rkpinctrl0
rkgpio3 at rkpinctrl0
rkgpio4 at rkpinctrl0
agtimer0 at mainbus0: tick rate 24000 KHz
dwge0 at mainbus0
dwge0: address: de:ad:be:ee:ee:ff
rgephy0 at dwge0 phy 0: RTL8169S/8110S/8211 PHY, rev. 6
dwmmc0 at mainbus0: 50 MHz base clock
sdmmc0 at dwmmc0: 4-bit, sd high-speed, mmc high-speed, dma
sdhc0 at mainbus0
sdhc0: SDHC 3.0, 200 MHz base clock
sdmmc1 at sdhc0: 8-bit, sd high-speed, mmc high-speed, dma
ehci0 at mainbus0
usb0 at ehci0: USB revision 2.0
uhub0 at usb0 configuration 1 interface 0 "Generic EHCI root hub" rev 2.00/1.00 addr 1
ehci1 at mainbus0
usb1 at ehci1: USB revision 2.0
uhub1 at usb1 configuration 1 interface 0 "Generic EHCI root hub" rev 2.00/1.00 addr 1
rkdwusb0 at mainbus0: "usb"
xhci0 at rkdwusb0, xHCI 1.16
usb2 at xhci0: USB revision 3.0
uhub2 at usb2 configuration 1 interface 0 "Generic xHCI root hub" rev 3.00/1.00 addr 1
rkdwusb1 at mainbus0: "usb"
xhci1 at rkdwusb1, xHCI 1.16
usb3 at xhci1: USB revision 3.0
uhub3 at usb3 configuration 1 interface 0 "Generic xHCI root hub" rev 3.00/1.00 addr 1
rkiic0 at mainbus0
iic0 at rkiic0
fanpwr0 at iic0 addr 0x40: SYR827, 1.00 VDC
fanpwr1 at iic0 addr 0x41: SYR828, 1.00 VDC
rkpmic0 at iic0 addr 0x1b: RK808
rkiic1 at mainbus0
iic1 at rkiic1
"everest,es8316" at iic1 addr 0x10 not configured
com0 at mainbus0: ns16550, no working fifo
com0: console
rktemp0 at mainbus0
rkiic2 at mainbus0
iic2 at rkiic2
"fairchild,fusb302" at iic2 addr 0x22 not configured
rkpcie0 at mainbus0
rkpcie0: link training timeout
cpu1 at mainbus0 mpidr 1: ARM Cortex-A53 r0p4
cpu2 at mainbus0 mpidr 2: ARM Cortex-A53 r0p4
cpu3 at mainbus0 mpidr 3: ARM Cortex-A53 r0p4
cpu4 at mainbus0 mpidr 100: ARM Cortex-A72 r0p2
cpu5 at mainbus0 mpidr 101: ARM Cortex-A72 r0p2
scsibus0 at sdmmc0: 2 targets, initiator 0
sd0 at scsibus0 targ 1 lun 0: <SD/MMC, USDU1, 0020> SCSI2 0/direct removable
sd0: 30429MB, 512 bytes/sector, 62318592 sectors
sdmmc1: can't enable card
vscsi0 at root
scsibus1 at vscsi0: 256 targets
softraid0 at root
scsibus2 at softraid0: 256 targets
bootfile: sd0a:/bsd
boot device: sd0
root on sd0a (5fc44b5ef06d6d53.a) swap on sd0b dump on sd0b
Automatic boot in progress: starting file system checks.
/dev/sd0a (5fc44b5ef06d6d53.a): file system is clean; not checking
/dev/sd0h (5fc44b5ef06d6d53.h): file system is clean; not checking
/dev/sd0j (5fc44b5ef06d6d53.j): file system is clean; not checking
/dev/sd0d (5fc44b5ef06d6d53.d): file system is clean; not checking
/dev/sd0f (5fc44b5ef06d6d53.f): file system is clean; not checking
/dev/sd0k (5fc44b5ef06d6d53.k): file system is clean; not checking
/dev/sd0g (5fc44b5ef06d6d53.g): file system is clean; not checking
/dev/sd0e (5fc44b5ef06d6d53.e): file system is clean; not checking
setting tty flags
pf enabled
starting network
dwge0: bound to 192.168.178.237 from 192.168.178.1 (de:ad:be:ee:ee:ff)
reordering libraries: done.
openssl: generating isakmpd/iked RSA keys... done.
ssh-keygen: generating new host keys: RSA DSA ECDSA ED25519 
starting early daemons: syslogd pflogd ntpd.
starting RPC daemons:.
savecore: no core dump
acpidump: Can't find ACPI information
checking quotas: done.
clearing /tmp
kern.securelevel: 0 -> 1
creating runtime link editor directory cache.
preserving editor files.
starting network daemons: sshd smtpd sndiod.
running rc.firsttime
Path to firmware: http://firmware.openbsd.org/firmware/snapshots/
No devices found which need firmware files to be downloaded.
starting local daemons: cron.
Wed Aug  8 08:19:08 CEST 2018

OpenBSD/arm64 (banff) (console)

login: 
```

## Caveats

The USB ports don't seem to work yet despite the controller attaching in OpenBSD, this is probably
due to the dtb not being fully correct. Also, the dtb should be built as part of the `sysutils/dtb`
port.
