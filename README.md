# Parallella RISC-V Prebuilt Images

## About this repository

This repository contains all the files needed to successfully boot RISC-V Linux and run RISC-V programs
on a RISC-V RV64IMA core that is placed inside the Zynq FPGA device of your Parallellla board.

All Parallella editions are supported:

* Desktop
* Microserver
* Embedded
* Kickstarter.

For more information visit the [Parallella RISC-V GSoC 2016 project repository](https://github.com/eliaskousk/parallella-riscv)

## Boot Files

Place the correct prebuilt boot files on the `boot` partition of your Parallella's SD card. Depending on
your Parallella edition you must copy them from one of the following boot folders:

* Parallella Desktop  / MicroServer Edition (Zynq 7010) - `boot.7010.rv64ima`
* Parallella Embedded / Kickstarter Edition (Zynq 7020) - `boot.7020.rv64ima`

After copying everything to your `boot` partition, you should have the following files:

File|Description
----|-----------
**devicetree.dtb**|Device Tree Blob
**parallella.bit.bin**|Parallella Base plus RISC-V RV64IMA Core
**uImage**|Zynq ARM Linux kernel

You can also use the Zynq ARM Linux kernel (uImage) from the Parallella E-SDK if you wish (only tested `v2016.3.1`).

### RISC-V Software

If you want to boot RISC-V Linux or run a simple baremetal program you must copy the `riscv` folder's
conents somewhere in your SD card's `root` filesystem, e.g in a new `/home/parallella/riscv` folder.

This can be done either by placing the SD card on your workstation and copying the files there or later
after the board is booted.

* File List

After copying everything to your riscv folder on the `root` partition, you should have the following files:

File|Description
----|-----------
**riscv/fesvr**|Frontend Server Application for HTIF transfers
**riscv/libfesvr.so**|Frontend Server Shared Library
**riscv/pk**|Proxy Kernel to load and support baremetal programs
**riscv/hello**|Hello World Program
**riscv/bbl**|Berkeley Boot Loader to boot and support RISC-V Linux
**riscv/vmlinux**|RISC-V Linux Kernel
**riscv/root.bin.tar.gz**|RISC-V Linux Root Filesystem Image Archive (Linaro 15.04)

* Copying software

To copy the built files to an already booted board you can run the following (assuming your Parallella
can be reached inside your network with the `parallella` host. You can replace this with parallella@ip
where ip is the IP address of Parallella in your local network):

```bash
scp riscv/fesvr parallella@parallella:~/riscv
scp riscv/libfesvr.so parallella@parallella:~/riscv
scp riscv/pk parallella@parallella:~/riscv
scp riscv/hello parallella@parallella:~/riscv
scp riscv/bbl parallella@parallella:~/riscv
scp riscv/vmlinux parallella@parallella:~/riscv
scp riscv/root.bin.tar.gz parallella@parallella:~/riscv
```

* Extra steps needed

The last file (root.bin.tar.gz) must be decompressed before using it to boot RISC-V Linux:

```bash
parallella@parallella:~/riscv/$ tar xzf root.bin.tar.gz

The root filesystem archive besides taking less space, it is also useful as a backup in case you
corrupt the root filesystem if you forget to shutdown the RISC-V Linux properly before powering off your board.

For the frontend server (fesvr) don't forget to copy the shared library `libfesvr.so` to `/usr/local/lib` on your
board and running `sudo ldconfig` to update the library cache.

```bash
parallella@parallella:~/riscv/$ sudo cp libfesvr.so /usr/local/lib/
parallella@parallella:~/riscv/$ sudo ldconfig
```

* Running baremetal programs

fesvr and pk can be used to load RISC-V baremetal programs like this:

```bash
parallella@parallella:~/$ sudo ./fesvr pk hello
Hello World!
```

* Booting RISC-V Linux

fesvr and bbl can be used to boot RISC-V Linux and perform simple tasks like shown below:

```bash
parallella@parallella:~/$ sudo ./fesvr +disk=root.bin bbl vmlinux
              vvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvv
                  vvvvvvvvvvvvvvvvvvvvvvvvvvvv
rrrrrrrrrrrrr       vvvvvvvvvvvvvvvvvvvvvvvvvv
rrrrrrrrrrrrrrrr      vvvvvvvvvvvvvvvvvvvvvvvv
rrrrrrrrrrrrrrrrrr    vvvvvvvvvvvvvvvvvvvvvvvv
rrrrrrrrrrrrrrrrrr    vvvvvvvvvvvvvvvvvvvvvvvv
rrrrrrrrrrrrrrrrrr    vvvvvvvvvvvvvvvvvvvvvvvv
rrrrrrrrrrrrrrrr      vvvvvvvvvvvvvvvvvvvvvv  
rrrrrrrrrrrrr       vvvvvvvvvvvvvvvvvvvvvv    
rr                vvvvvvvvvvvvvvvvvvvvvv      
rr            vvvvvvvvvvvvvvvvvvvvvvvv      rr
rrrr      vvvvvvvvvvvvvvvvvvvvvvvvvv      rrrr
rrrrrr      vvvvvvvvvvvvvvvvvvvvvv      rrrrrr
rrrrrrrr      vvvvvvvvvvvvvvvvvv      rrrrrrrr
rrrrrrrrrr      vvvvvvvvvvvvvv      rrrrrrrrrr
rrrrrrrrrrrr      vvvvvvvvvv      rrrrrrrrrrrr
rrrrrrrrrrrrrr      vvvvvv      rrrrrrrrrrrrrr
rrrrrrrrrrrrrrrr      vv      rrrrrrrrrrrrrrrr
rrrrrrrrrrrrrrrrrr          rrrrrrrrrrrrrrrrrr
rrrrrrrrrrrrrrrrrrrr      rrrrrrrrrrrrrrrrrrrr
rrrrrrrrrrrrrrrrrrrrrr  rrrrrrrrrrrrrrrrrrrrrr

       INSTRUCTION SETS WANT TO BE FREE
[    0.000000] Linux version 4.1.17-g174f395-dirty (lupo@gene) (gcc version 5.3.0 (GCC) ) #9 Mon Jul 25 18:18:14 EEST 2016
[    0.000000] Available physical memory: 1022MB
[    0.000000] Physical memory usage limited to 384MB
[    0.000000] Zone ranges:
[    0.000000]   Normal   [mem 0x0000000000200000-0x00000000181fffff]
[    0.000000] Movable zone start for each node
[    0.000000] Early memory node ranges
[    0.000000]   node   0: [mem 0x0000000000200000-0x00000000181fffff]
[    0.000000] Initmem setup node 0 [mem 0x0000000000200000-0x00000000181fffff]
[    0.000000] Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 96960
[    0.000000] Kernel command line: root=/dev/htifblk0 mem=384M
[    0.000000] PID hash table entries: 2048 (order: 2, 16384 bytes)
[    0.000000] Dentry cache hash table entries: 65536 (order: 7, 524288 bytes)
[    0.000000] Inode-cache hash table entries: 32768 (order: 6, 262144 bytes)
[    0.000000] Sorting __ex_table...
[    0.000000] Memory: 384352K/393216K available (1810K kernel code, 92K rwdata, 396K rodata, 64K init, 210K bss, 8864K reserved, 0K cma-reserved)
[    0.000000] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
[    0.000000] NR_IRQS:2
[    0.000000] clocksource riscv_clocksource: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 191126044627 ns
[    0.150000] Calibrating delay using timer specific routine.. 20.02 BogoMIPS (lpj=100104)
[    0.150000] pid_max: default: 32768 minimum: 301
[    0.150000] Mount-cache hash table entries: 1024 (order: 1, 8192 bytes)
[    0.150000] Mountpoint-cache hash table entries: 1024 (order: 1, 8192 bytes)
[    0.150000] devtmpfs: initialized
[    0.150000] clocksource jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 19112604462750000 ns
[    0.150000] NET: Registered protocol family 16
[    0.150000] Switched to clocksource riscv_clocksource
[    0.150000] NET: Registered protocol family 2
[    0.150000] TCP established hash table entries: 4096 (order: 3, 32768 bytes)
[    0.150000] TCP bind hash table entries: 4096 (order: 3, 32768 bytes)
[    0.150000] TCP: Hash tables configured (established 4096 bind 4096)
[    0.150000] UDP hash table entries: 256 (order: 1, 8192 bytes)
[    0.150000] UDP-Lite hash table entries: 256 (order: 1, 8192 bytes)
[    0.150000] NET: Registered protocol family 1
[    0.150000] futex hash table entries: 256 (order: 0, 6144 bytes)
[    0.150000] io scheduler noop registered
[    0.150000] io scheduler cfq registered (default)
[    0.180000] htifcon htif1: detected console
[    0.180000] console [htifcon0] enabled
[    0.180000] htifblk htif2: detected disk
[    0.180000] htifblk htif2: added htifblk0
[    0.180000] VFS: Mounted root (ext2 filesystem) readonly on device 254:0.
[    0.180000] devtmpfs: mounted
[    0.180000] Freeing unused kernel memory: 64K (ffffffff80000000 - ffffffff80010000)
INIT: version 2.88 booting
[    0.430000] random: dd urandom read with 23 bits of entropy available
hwclock: can't open '/dev/misc/rtc': No such file or directory
Sun Jul 17 01:37:12 UTC 2016
hwclock: can't open '/dev/misc/rtc': No such file or directory
INIT: Entering runlevel: 5
Configuring network interfaces... ifconfig: SIOCGIFFLAGS: No such device
hwclock: can't open '/dev/misc/rtc': No such file or directory
Starting syslogd/klogd: done

Poky (Yocto Project Reference Distro) 2.0+snapshot-20160717 riscv64 /dev/ttyHTIF0

riscv64 login: root

root@riscv64:~# ls -al /
drwxr-xr-x   18 root     root          1024 Jul 17 01:37 .
drwxr-xr-x   18 root     root          1024 Jul 17 01:37 ..
drwxr-xr-x    2 root     root          2048 Jul 17 01:37 bin
drwxr-xr-x    2 root     root          1024 Jul 17 01:17 boot
drwxr-xr-x    3 root     root         10720 Jul 17 01:37 dev
drwxr-xr-x   21 root     root          1024 Jan  1  1970 etc
drwxr-xr-x    3 root     root          1024 Jul 17 01:37 home
drwxr-xr-x    2 root     root          2048 Jul 17 01:37 lib
drwx------    2 root     root         12288 Jul 17 01:37 lost+found
drwxr-xr-x    2 root     root          1024 Jul 17 01:17 media
drwxr-xr-x    2 root     root          1024 Jul 17 01:17 mnt
dr-xr-xr-x   29 root     root             0 Jan  1  1970 proc
drwxr-xr-x    3 root     root           160 Jul 17 01:37 run
drwxr-xr-x    2 root     root          1024 Jul 17 01:37 sbin
drwxr-xr-x    2 root     root          1024 Jul 17 01:17 sys
drwxrwxrwt    2 root     root          1024 Jul 17 01:17 tmp
drwxr-xr-x    9 root     root          1024 Jul 17 01:34 usr
drwxr-xr-x    8 root     root          1024 Jul 17 01:34 var

root@riscv64:~# uname -a
Linux riscv64 4.1.17-g174f395-dirty #9 Mon Jul 25 18:18:14 EEST 2016 riscv GNU/Linux

root@riscv64:~# shutdown -h now
INIT: Switching to runlevel: 0
INIT: Sending processes the TERM signal
root@riscv64:~# logout
hwclock: can't open '/dev/misc/rtc': No such file or directory
Stopping syslogd/klogd: stopped syslogd (pid 142)
stopped klogd (pid 141)
done
Deconfiguring network interfaces... ifdown: interface eth0 not configured
done.
Sending all processes the TERM signal...
Unmounting remote filesystems...
Deactivating swap...
Unmounting local filesystems...
[4.730000] reboot: Power down

parallella@parallella:~/$
```

Congratulations, you now have a working RISC-V RV64 core running RISC-V Linux on your Parallella!

## Links

[Parallella RISC-V GSoC 2016 Project](https://github.com/eliaskousk/parallella-riscv)

[FOSSi](http://www.fossi-foundation.org)

[Parallella](https://www.parallella.org)

[RISC-V](http://riscv.org)

[Google Summer of Code](https://developers.google.com/open-source/gsoc/)

[FOSSi GSoC'16 Projects](https://summerofcode.withgoogle.com/organizations/5516229267685376/#projects)

## Contributors

### Code

- Elias Kouskoumvekakis ([Blog](http://eliaskousk.teamdac.com))

### GSoC Mentors

- Olof    Kindgren ([FOSSi Foundation](http://fossi-foundation.org))
- Andreas Olofsson ([Adapteva](http://www.adapteva.com))
