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


The bootlogs i captured. Actually i unplugged it accidentally while it was updating its firmware (luckily it was already almost finished) and now it spams 'the ispfile not ready, wait' and after a few seconds the watchdog reboots the camera. Once i crack the root password i can maybe use the Tapo_kill.sh script to end all the applications and then debug the filesystem and disable the watchdog so i can connect it to the app without it resetting itself before the app can connect to it so i can hopefully let it install the firmware correctly! The bootlogs is in a seperate file i uploaded because github messed up the bootargs in the readme so i did them seperately


There is a way to enter the boot loader shell. if it says "Autobooting in 1 second" inmideatly enter 'slp' in the shell. This will halt the bot process and allow you to view the environmental variables.
Idk how but i kind of managed to f it up and now the original bootcmd does not work anymore:

'bootcmd=echo read normal boot at 0x80200000'

But when resetting the environment to default using 'env default -f -a' i got an alternative bootcmd that works (it fully works and i tested it before the firmware update that failed):

'bootcmd=sf probe; sf read 0x80700000 0x80200 0x175000; bootm 0x80700000'

Original Bootargs:

'bootargs=console=ttyS1,115200n8 mem=42M@0x0 rmem=22M@0x2a00000 root=/dev/mtdblock6 rootfstype=squashfs spdev=/dev/mtdblock7 noinitrd init=/etc/preinit'


I then decided to see if i can get a shell and bypass the pasword protection. I patched the boot args to use /bin/sh as Init instead of /etc/preinit. This worked and i got a shell. Now i only had to mount /proc /dev and /sys but somehow running /etc/preinit worked and it did actually not start the whole application stack but just mounted everything. Now i had /proc and /dev and /sys populated most commands worked with no issue. I also managed to dump the etc/passwd file and got a password hash. Patched bootargs here:

'bootargs=console=ttyS1,115200n8 mem=42M@0x0 rmem=22M@0x2a00000 root=/dev/mtdblock6 rootfstype=squashfs spdev=/dev/mtdblock7 noinitrd init=/bin/sh'

Password hash in /etc/passwd. There seem to be more users but they are turned off i think. The main account is admin. It seems you only have to put in a password because when clicking enter it prompts me with a 'C200 Login':

root:$1$7rMT0TnJ$e/hFcoKvc.N4w8uryOtlg/:0:0:root:/root:/bin/ash
nobody:*:65534:65534:nobody:/var:/bin/false
admin:*:500:500:admin:/var:/bin/false
guest:*:500:500:guest:/var:/bin/false
ftp:*:55:55:ftp:/home/ftp:/bin/false

I sadly did not manage to crack the password yet. I used hashcat and jonh and no dice! it has nothing to do with Slp or any of the other key words that worked on older v1 and v2 models that used realtek soc's. I am now trying to search through any GPL sources hoping i could find the password or how the password was generated (with a bit of luck they left some password encryption binary into the gpl sources that allows me to create and compare hashes). I might now try to replace the password and make my own hash but i am still trying to figure out how that is done cause i never done this before. I might even try to put the rootfs on an sd card and to let the kernel on the flash boot from it. I did upload the binary just now!

 Fun fact the firmware is based on Openwrt because i saw multiple references to it. Its just that TPLink added some custom binaries to it. They provide the build files and it can be made via buildroot but you need to specify a variable named PR_NAME= which i think stands for productname. 

I am now investigating it further to find a way to force it to update its firmware so i have the complete non corrupted firmware which i than can fully inspect. I also want to try and make my own squashfs fileststem with my own root password set as 'root' simply. I could not find any binary that created the password so i bet TPLink compiled it in in the buildroot process and its hardvoded in the etc/passwd file










