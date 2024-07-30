[passwd.txt](https://github.com/user-attachments/files/16426704/passwd.txt)[bootargs_patched.txt](https://github.com/user-attachments/files/16426675/bootargs_patched.txt)[new_bootcmd.txt](https://github.com/user-attachments/files/16426651/new_bootcmd.txt)[default_bootcmd.txt](https://github.com/user-attachments/files/16426635/default_bootcmd.txt)[bootlogs.txt](https://github.com/user-attachments/files/16426597/bootlogs.txt)[bootlogs.txt](https://github.com/user-attachments/files/16426533/bootlogs.txt)# Tapo-C200V3-Cam-Research-Project

In this repository, I will post all the information I gathered while opening a TP-Link Tapo C200 version 3 WiFi camera and I will share the progress and findings.

## License

This project is licensed under the GNU General Public License (GPL) Version 3. See the LICENSE file for more information. The reason for this is that in this repo, I might share some device components that are licensed under GPLv3. Examples include: The Linux kernel 3.10.14 modified for Ingenic by TP-Link, Busybox, U-Boot, filesystem formats, drivers, and kernel modules. Additionally, I might share the SDK of the device.

## For the Manufacturer

If you are the manufacturer (TP-Link) or a third party who worked on this product (the TP-Link Tapo C200 version 3 camera) and wish to have this page removed, feel free to contact me via email at 309electronics@gmail.com and I will happily cooperate! I do not want to break any laws because I am just a hobbyist who likes sharing his passion for hardware tinkering/hacking and electronics. I do not want to get into any legal trouble, so if you wish for this page to be removed, I will happily cooperate and remove this page to avoid any legal issues. :) Please feel free to alert/contact me about any violations if there are any!

## Disclaimer

The use of these files and information is at your own risk! Make sure to respect the rules of the manufacturer and the GPL, and that you understand the possible legal implications when using the files.

## Story

A few months back, I got into hardware hacking/tinkering because I saw a few videos of Matt Brown on YouTube who does exactly this: hacking hardware and protocols. So, I wanted to try it myself. I was already familiar with the UART protocol and bootlogs, and even tried to unbrick a router (which sadly failed). I also hooked up UART to a KPN (Dutch provider) TV box and saw bootlogs that got me engaged in diving deeper.

I am a hobbyist, so I thought it would be fun to dive deeper into hardware hacking and start by tinkering with device firmware. I knew these embedded devices usually run Linux with Busybox and a custom application stack containing the scripts and applications from the manufacturer that handle the job the product was designed for. I was also familiar with a few common embedded bootloaders like Broadcom's CFE, the Realtek bootloader, and of course one of the most popular ones, Das-U-Boot (or U-Boot for short).

I have already messed with tons of devices, including another camera, but that is too much to name it all. I saw a GitHub post for this camera (and a firmware site for all revisions from v1 to v4) which ended up being a different version that had a Realtek SoC but still ran U-Boot. So, I bought the camera not knowing my camera was different yet in terms of hardware and root password.

When I opened it up (almost immediately after it arrived at my door), I saw that it had different hardware. This camera had a whole different beast of SoC from the Realtek in the other version(s). It actually had the Ingenic T31, which I was also pretty familiar with because I saw it in an LSC solar camera which I bought at Action (a store in Germany/the Netherlands) which I believe I also have a repo of. This chip is quite a beast! It contains not only a MIPS 32-bit 1.5GHz CPU which Ingenic calls the XBurst core, it also contains a RISC-V microcontroller core running at 500MHz and some image signal processor and video processor. The MIPS runs the OS, which can be Linux, while the RISC-V runs an RTOS (real-time operating system) and can be used for smart functions like motion detection, CPU sleep mode, watchdog, etc. So, I was kind of happy seeing a pretty powerful but common friend inside!


The backside of the board which you can see when you take the top cover off:
![1000144356](https://github.com/user-attachments/assets/c6e4cee3-9c50-4893-be3d-abfd11f0ee60)

The frontside of the board with the sd card holder:
![1000144357](https://github.com/user-attachments/assets/1bd5ec8d-7043-4399-8005-8b2ebd4ba042)


The Ingenic T31 Soc With the block diagram showing the cores and chip elements:
![IMG20240730123311](https://github.com/user-attachments/assets/6c22a935-b956-4bcf-8d64-7f0c0d113c06)
Block diagram:
![Ingenic-T31-MIPS-RISC-V-Video-Processor](https://github.com/user-attachments/assets/5c20e486-ac17-4bf4-b240-1351b4af31da)


The Realtek 8188FTV wireless chip. (This chip actually comminucates over usb so might also be an interesting target. This chip (or clones of it) is often seen in these IOT cameras because its cheap and works fine. They are also often on a module/daughterboard but that is not the case here)
![IMG20240730123847](https://github.com/user-attachments/assets/e714b2dc-4498-4f04-8fd6-d562469d493d)


There also seems to be a mysterious spot for a usb port... Maybe for usb variants 🤔?????
![1000144352](https://github.com/user-attachments/assets/03d8e03e-d62f-42f4-abad-5aa3c1048393)


The UTC2803M current mode PWM driver (used for the stepper motors and the PTZ functions)
![1000144350](https://github.com/user-attachments/assets/0de38db3-e698-4f3b-85c8-72508059fa0f)


The XMC XM25QH64C flash chip. This is 64 megabits / 8 thus 8 megabytes of flash. Sufficient for storing the bootloader and Linux os.
![1000144351](https://github.com/user-attachments/assets/7ab30ed1-dfbc-4cfa-89cb-125c0069ec55)


The 2 pads i had to short to enable the Uart port (I dont know if this is needed on all models). I did a pretty horrible solder job here but hey, it works!
![1000144354](https://github.com/user-attachments/assets/0d8e54bf-fd07-4d85-a0e3-a3fc26ff1d37)


The Uart port (Labels for the pinout is next to the stepper motor connector and reads Tx, Rx, GND, vcc) Only Tx, Rx, GND are needed. DO NOT connect vcc or it might damage your camera or usb to serial. Also connect Tx of the camera to Rx of the usb to uart (This will be the line that allows the camera to talk to the usb to uart and thus your terminal session), Rx of the camera to Tx of the usb 2 uart (This allows the Terminal session on the host computer to talk to the camera via the usb to uart), and Gnd to Gnd. It is advised to put some glue over the pads like hotglue to make sure you dont rip the pads of the board which happened to me with earlier devices i messed with. here i also forgot it because i ran out of hot glue.:
![1000144353 (1)](https://github.com/user-attachments/assets/0ba6a419-e921-4232-907d-10e0311594bd)


The bootlogs i captured. Actually i unplugged it accidentally while it was updating its firmware (luckily it was already almost finished) and now it spams 'the ispfile not ready, wait' and after a few seconds the watchdog reboots the camera. Once i crack the root password i can maybe use the Tapo_kill.sh script to end all the applications and then debug the filesystem and disable the watchdog so i can connect it to the app without it resetting itself before the app can connect to it so i can hopefully let it install the firmware correctly!:

[Uploading bootlU-Boot SPL 2013.07 (Apr 26 2023 - 10:09:24)
Timer init
CLK stop
PLL init
pll_init:366
pll_cfg.pdiv = 10, pll_cfg.h2div = 5, pll_cfg.h0div = 5, pll_cfg.cdiv = 1, pll_cfg.l2div = 2
nf=118 nr = 1 od0 = 1 od1 = 2
cppcr is 07605100
CPM_CPAPCR 0750510d
nf=84 nr = 1 od0 = 1 od1 = 2
cppcr is 05405100
CPM_CPMPCR 07d0590d
nf=100 nr = 1 od0 = 1 od1 = 2
cppcr is 06405100
CPM_CPVPCR 0640510d
cppcr 0x9a773310
apll_freq 1404000000
mpll_freq 1000000000
vpll_freq = 1200000000
ddr sel mpll, cpu sel apll
ddrfreq 500000000
cclk  1404000000
l2clk 702000000
h0clk 200000000
h2clk 200000000
pclk  100000000
CLK init
SDRAM init
sdram init start
ddr_inno_phy_init ..!
phy reg = 0x00000007, CL = 0x00000007
ddr_inno_phy_init ..! 11:  00000004
ddr_inno_phy_init ..! 22:  00000006
ddr_inno_phy_init ..! 33:  00000006
REG_DDR_LMR: 00000210
REG_DDR_LMR: 00000310
REG_DDR_LMR: 00000110
REG_DDR_LMR, MR0: 00f73011
T31_0x5: 00000007
T31_0x15: 0000000c
T31_0x4: 00000000
T31_0x14: 00000002
INNO_TRAINING_CTRL 1: 00000000
INNO_TRAINING_CTRL 2: 000000a1
T31_cc: 00000003
INNO_TRAINING_CTRL 3: 000000a0
T31_118: 0000003c
T31_158: 0000003c
T31_190: 00000021
T31_194: 0000001f
jz-04 :  0x00000051
jz-08 :  0x000000a0
jz-28 :  0x00000024
DDR PHY init OK
INNO_DQ_WIDTH   :00000003
INNO_PLL_FBDIV  :00000014
INNO_PLL_PDIV   :00000005
INNO_MEM_CFG    :00000051
INNO_PLL_CTRL   :00000018
INNO_CHANNEL_EN :0000000d
INNO_CWL        :00000006
INNO_CL         :00000007
DDR Controller init
DDRC_STATUS         0x80000001
DDRC_CFG            0x0a288a40
DDRC_CTRL           0x0000011c
DDRC_LMR            0x00400008
DDRC_DLP            0x00000000
DDRC_TIMING1        0x040e0806
DDRC_TIMING2        0x02170707
DDRC_TIMING3        0x2007051e
DDRC_TIMING4        0x1a240031
DDRC_TIMING5        0xff060405
DDRC_TIMING6        0x32170505
DDRC_REFCNT         0x00f26801
DDRC_MMAP0          0x000020fc
DDRC_MMAP1          0x00002400
DDRC_REMAP1         0x03020d0c
DDRC_REMAP2         0x07060504
DDRC_REMAP3         0x0b0a0908
DDRC_REMAP4         0x0f0e0100
DDRC_REMAP5         0x13121110
DDRC_AUTOSR_EN      0x00000000
sdram init finished
SDRAM init ok
board_init_r
image entry point: 0x80100000


U-Boot 2013.07 (Apr 26 2023 - 10:09:24)

Board: ISVP (Ingenic XBurst T31 SoC)
DRAM:  64 MiB
Top of RAM usable for U-Boot at: 84000000
Reserving 425k for U-Boot at: 83f94000
Reserving 32784k for malloc() at: 81f90000
Reserving 32 Bytes for Board Info at: 81f8ffe0
Reserving 124 Bytes for Global Data at: 81f8ff64
Reserving 128k for boot params() at: 81f6ff64
Stack Pointer at: 81f6ff48
Now running in RAM - U-Boot at: 83f94000
MMC:   msc: 0
the id code = 204017
unsupport ID is if the id not be 0x00,the flash is ok for burner
the manufacturer 20
SF: Detected SK25Q128

In:    serial
Out:   serial
Err:   serial
Net:   No ethernet found.
Autobooting in 1 seconds
the id code = 204017
unsupport ID is if the id not be 0x00,the flash is ok for burner
the manufacturer 20
SF: Detected SK25Q128

--->probe spend 11 ms
SF: 1527808 bytes @ 0x80200 Read: OK
--->read spend 493 ms
## Booting kernel from Legacy Image at 80700000 ...
   Image Name:   Linux-3.10.14__isvp_swan_1.0__
   Image Type:   MIPS Linux Kernel Image (lzma compressed)
   Data Size:    1263632 Bytes = 1.2 MiB
   Load Address: 80010000
   Entry Point:  803076a0
   Verifying Checksum ... OK
   Uncompressing Kernel Image ... OK

Starting kernel ...

[    0.000000] Initializing cgroup subsys cpu
[    0.000000] Initializing cgroup subsys cpuacct
[    0.000000] Linux version 3.10.14__isvp_swan_1.0__ (root@smartlifeci1) (gcc version 4.7.2 (Ingenic r2.3.3 2016.12) ) #2 PREEMPT Mon May 13 12:07:09 CST 2024
[    0.000000] bootconsole [early0] enabled
[    0.000000] CPU0 RESET ERROR PC:E3090118
[    0.000000] CPU0 revision is: 00d00100 (Ingenic Xburst)
[    0.000000] FPU revision is: 00b70000
[    0.000000] CCLK:1404MHz L2CLK:702Mhz H0CLK:250MHz H2CLK:250Mhz PCLK:125Mhz
[    0.000000] Determined physical RAM map:
[    0.000000]  memory: 00399000 @ 00010000 (usable)
[    0.000000]  memory: 00037000 @ 003a9000 (usable after init)
[    0.000000] User-defined physical RAM map:
[    0.000000]  memory: 02d00000 @ 00000000 (usable)
[    0.000000] Zone ranges:
[    0.000000]   Normal   [mem 0x00000000-0x02cfffff]
[    0.000000] Movable zone start for each node
[    0.000000] Early memory node ranges
[    0.000000]   node   0: [mem 0x00000000-0x02cfffff]
[    0.000000] Primary instruction cache 32kB, 8-way, VIPT, linesize 32 bytes.
[    0.000000] Primary data cache 32kB, 8-way, VIPT, no aliases, linesize 32 bytes
[    0.000000] pls check processor_id[0x00d00100],sc_jz not support!
[    0.000000] MIPS secondary cache 128kB, 8-way, linesize 32 bytes.
[    0.000000] Built 1 zonelists in Zone order, mobility grouping off.  Total pages: 11430
[    0.000000] Kernel command line: console=ttyS1,115200n8 mem=45M@0x0 rmem=19M@0xa00000 root=/dev/mtdblock6 rootfstype=squashfs spdev=/dev/mtdblock7 noinitrd init=/etc/preinit
[    0.000000] PID hash table entries: 256 (order: -2, 1024 bytes)
[    0.000000] Dentry cache hash table entries: 8192 (order: 3, 32768 bytes)
[    0.000000] Inode-cache hash table entries: 4096 (order: 2, 16384 bytes)
[    0.000000] Memory: 41124k/46080k available (3070k kernel code, 4956k reserved, 613k data, 220k init, 0k highmem)
[    0.000000] SLUB: HWalign=32, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
[    0.000000] Preemptible hierarchical RCU implementation.
[    0.000000] NR_IRQS:358
[    0.000000] clockevents_config_and_register success.
[    0.000015] Calibrating delay loop... 1397.55 BogoMIPS (lpj=6987776)
[    0.037793] pid_max: default: 32768 minimum: 301
[    0.042653] Mount-cache hash table entries: 512
[    0.047599] Initializing cgroup subsys debug
[    0.051856] Initializing cgroup subsys freezer
[    0.057347] devtmpfs: initialized
[    0.061472] regulator-dummy: no parameters
[    0.065720] NET: Registered protocol family 16
[    0.072290] bio: create slab <bio-0> at 0
[    0.077768] jz-dma jz-dma: JZ SoC DMA initialized
[    0.082657] usbcore: registered new interface driver usbfs
[    0.088240] usbcore: registered new interface driver hub
[    0.093664] usbcore: registered new device driver usb
[    0.098844]  (null): set:311  hold:312 dev=125000000 h=625 l=625
[    0.105753] Switching to clocksource jz_clocksource
[    0.110814] cfg80211: Calling CRDA to update world regulatory domain
[    0.117785] jz-dwc2 jz-dwc2: cgu clk gate get error
[    0.122694] DWC IN OTG MODE
[    0.126114] dwc2 dwc2: Keep PHY ON
[    0.129478] dwc2 dwc2: Using Buffer DMA mode
[    0.133828] dwc2 dwc2: Core Release: 3.00a
[    0.137988] dwc2 dwc2: DesignWare USB2.0 High-Speed Host Controller
[    0.144334] dwc2 dwc2: new USB bus registered, assigned bus number 1
[    0.151206] usb usb1: New USB device found, idVendor=1d6b, idProduct=0002
[    0.158014] usb usb1: New USB device strings: Mfr=3, Product=2, SerialNumber=1
[    0.165374] usb usb1: Manufacturer: Linux 3.10.14__isvp_swan_1.0__ dwc2-hcd
[    0.172414] usb usb1: SerialNumber: dwc2
[    0.176674] hub 1-0:1.0: USB hub found
[    0.180405] hub 1-0:1.0: 1 port detected
[    0.184571] dwc2 dwc2: DWC2 Host Initialized
[    0.188972] NET: Registered protocol family 2
[    0.193777] TCP established hash table entries: 512 (order: 0, 4096 bytes)
[    0.200750] TCP bind hash table entries: 512 (order: -1, 2048 bytes)
[    0.207129] TCP: Hash tables configured (established 512 bind 512)
[    0.213450] TCP: reno registered
[    0.216662] UDP hash table entries: 256 (order: 0, 4096 bytes)
[    0.222606] UDP-Lite hash table entries: 256 (order: 0, 4096 bytes)
[    0.229122] NET: Registered protocol family 1
[    0.233776] freq_udelay_jiffys[0].max_num = 10
[    0.238212] dwc2 dwc2: ID PIN CHANGED!
[    0.242080] cpufreq  udelay  loops_per_jiffy
[    0.246415] 12000     59724   59724
[    0.249681] 24000     119449  119449
[    0.253136] 60000     298622  298622
[    0.256568] 120000    597245  597245
[    0.260100] 200000    995409  995409
[    0.263639] 300000    1493114         1493114
[    0.267340] 600000    2986229         2986229
[    0.271055] 792000    3941822         3941822
[    0.274756] 1008000   5016864         5016864
[    0.278553] 1200000   5972458         5972458
[    0.286012] squashfs: version 4.0 (2009/01/31) Phillip Lougher
[    0.292087] msgmni has been set to 80
[    0.296675] io scheduler noop registered
[    0.300598] io scheduler cfq registered (default)
[    0.307206] jz-uart.1: ttyS1 at MMIO 0x10031000 (irq = 58) is a uart1
[    0.314808] console [ttyS1] enabled, bootconsole disabled
[    0.314808] console [ttyS1] enabled, bootconsole disabled
[    0.326520] zram: Created 2 device(s) ...
[    0.330884] logger: created 256K log 'log_main'
[    0.336007] jz SADC driver registeres over!
[    0.341538] jz TCU driver register completed
[    0.346236] the id code = 204017, the flash name is XM25QH64C
[    0.352399] JZ SFC Controller for SFC channel 0 driver register
[    0.358521] SLP flash nor read
[    0.361806] Searching for RedBoot partition table
[    0.366680] 10 RedBoot partitions found on MTD device jz_sfc
[    0.372547] Creating 10 MTD partitions on "jz_sfc":
[    0.377598] 0x000000000000-0x00000002d800 : "factory_boot"
[    0.383290] mtd: partition "factory_boot" doesn't end on an erase block -- force read-only
[    0.392341] 0x00000002d800-0x000000030000 : "factory_info"
[    0.398017] mtd: partition "factory_info" doesn't start on an erase block boundary -- force read-only
[    0.408032] 0x000000030000-0x000000040000 : "art"
[    0.413384] 0x000000040000-0x000000060000 : "config"
[    0.418958] 0x000000060000-0x000000080000 : "normal_boot"
[    0.425068] 0x000000080000-0x0000001b5200 : "kernel"
[    0.430208] mtd: partition "kernel" doesn't end on an erase block -- force read-only
[    0.438757] 0x0000001b5200-0x000000440000 : "rootfs"
[    0.443931] mtd: partition "rootfs" doesn't start on an erase block boundary -- force read-only
[    0.453398] 0x000000440000-0x0000007ffe00 : "rootfs_data"
[    0.458986] mtd: partition "rootfs_data" doesn't end on an erase block -- force read-only
[    0.467918] 0x0000007ffe00-0x000000800000 : "verify"
[    0.473132] mtd: partition "verify" doesn't start on an erase block boundary -- force read-only
[    0.482602] 0x000000080000-0x000000800000 : "firmware"
[    0.488340] SPI NOR MTD LOAD OK
[    0.491684] tun: Universal TUN/TAP device driver, 1.6
[    0.496908] tun: (C) 1999-2004 Max Krasnyansky <maxk@qualcomm.com>
[    0.503486] usbcore: registered new interface driver asix
[    0.509098] usbcore: registered new interface driver ax88179_178a
[    0.515534] usbcore: registered new interface driver cdc_ether
[    0.521636] usbcore: registered new interface driver net1080
[    0.527523] usbcore: registered new interface driver cdc_subset
[    0.533707] usbcore: registered new interface driver zaurus
[    0.539527] usbcore: registered new interface driver cdc_ncm
[    0.545824] usbcore: registered new interface driver usbhid
[    0.551630] usbhid: USB HID core driver
[    0.555723] Registered character driver slp_flash_chrdev
[    0.561236] SLP flash cdev init!  jz_sfc
[    0.565286] TCP: cubic registered
[    0.568894] NET: Registered protocol family 17
[    0.574066] /home/jenkins/tapo/c200v3_develop/slp-sp-target-src/ingenict31/linux-3.10.14/drivers/rtc/hctosys.c: unable to open rtc device (rtc0)
[    0.592296] VFS: Mounted root (squashfs filesystem) readonly on device 31:6.
[    0.602915] devtmpfs: mounted
[    0.606388] Freeing unused kernel memory: 220K (803a9000 - 803e0000)
[    0.700725] usb 1-1: new high-speed USB device number 2 using dwc2
[    0.911254] usb 1-1: New USB device found, idVendor=0bda, idProduct=f179
[    0.922615] usb 1-1: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[    0.939163] usb 1-1: SerialNumber: 98254A3608BB
- preinit -
mounting rwroot
- init -
[    1.902832] jzmmc_v1.2 jzmmc_v1.2.0: vmmc regulator missing
[    1.981679] jzmmc_v1.2 jzmmc_v1.2.0: register success!

Please press Enter to activate this console. [    2.130769] zram0: detected capacity change from 0 to 16777216
Setting up swapspace version 1, size = 16773120 bytes
UUID=ecdd7cc5-45bf-4e24-8d67-1[    2.143323] Adding 16380k swap on /dev/zram0.  Priority:-1 extents:1 across:16380k SS
177cc12b198
Mon May 13 04:06:58 GMT 2024
ispfile is not ready, wait...
ispfile is not ready, wait...
ispfile is not ready, wait...
ogs.txt…]()


There is a way to enter the boot loader shell. if it says "Autobooting in 1 second" inmideatly enter 'slp' in the shell. This will halt the bot process and allow you to view the environmental variables.
Idk how but i kind of managed to f it up and now the original bootcmd does not work anymore:
[Uploading default_bootcmd=echo read normal boot at 0x80200000
bootcmd.txt…]()

 But when resetting the environment to default using 'env default -f -a' i got an alternative bootcmd that works (it fully works and i tested it before the firmware update that failed):
 [Uploading new_bootcmd.tsf probe; sf read 0x80700000 0x80200 0x175000; bootm 0x80700000xt…]()

Original Bootargs:
[bootargs.txt](https://github.com/user-attachments/files/16426657/bootargs.txt)bootargs=console=ttyS1,115200n8 mem=42M@0x0 rmem=22M@0x2a00000 root=/dev/mtdblock6 rootfstype=squashfs spdev=/dev/mtdblock7 noinitrd init=/etc/preinit



I then decided to see if i can get a shell and bypass the pasword protection. I patched the boot args to use /bin/sh as Init instead of /etc/preinit. This worked and i got a shell. Now i only had to mount /proc /dev and /sys but somehow running /etc/preinit worked and it did actually not start the whole application stack but just mounted everything. Now i had /proc and /dev and /sys populated most commands worked with no issue. I also managed to dump the etc/passwd file and got a password hash:
[Uploading bootargs_patchebootargs=console=ttyS1,115200n8 mem=42M@0x0 rmem=22M@0x2a00000 root=/dev/mtdblock6 rootfstype=squashfs spdev=/dev/mtdblock7 noinitrd init=/bin/shd.txt…]()

Password hash in /etc/passwd. There seem to be more users but they are turned off i think. The main account is admin. It seems you only have to put in a password because when clicking enter it prompts me with a 'C200 Login':
 [Uploading passwroot:$1$7rMT0TnJ$e/hFcoKvc.N4w8uryOtlg/:0:0:root:/root:/bin/ash
nobody:*:65534:65534:nobody:/var:/bin/false
admin:*:500:500:admin:/var:/bin/false
guest:*:500:500:guest:/var:/bin/false
ftp:*:55:55:ftp:/home/ftp:/bin/false
d.txt…]()

I am now Trying to crack the password hash and will update you when i found the password if its possible at all......










