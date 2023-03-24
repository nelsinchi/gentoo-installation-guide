# Gentoo Linux - Customized Installation Guide
# NOTE: The guide is still underconstruction...

**Steps to Cover**
```
1- Booting
2- Optional: User accounts
3- Optional: Viewing documentation while installing
4- Optional: Starting the SSH daemon
5- Connecting wireless internet
```

# Booting

https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Media

# Optional: User accounts

If other people need access to the installation environment, or there is need to run commands as a non-root user on the installation medium (such as to chat using **irssi** without root privileges for security reasons), then an additional user account needs to be created and the root password set to a strong password.

To change the root password, use the **passwd** utility:

```
livecd ~ # passwd
New password: (Enter the new password)
Re-enter password: (Re-enter the password)
```

To create a user account, first enter their credentials, followed by the account's password. The **useradd** and **passwd** commands are used for these tasks.

In the next example, a user called john is created:

```
livecd ~ # useradd -m -G users nelson
livecd ~ # passwd nelson
New password: (Enter nelson's password)
Re-enter password: (Re-enter nelson's password)
```

# Optional: Viewing documentation while installing

**TTYs**
To view the Gentoo handbook during the installation, first create a user account as described above. Then press `Alt+F2` to go to a new terminal.

During the installation, the **links** command can be used to browse the Gentoo handbook - of course only from the moment that the Internet connection is working.

```
livecd ~ # links https://wiki.gentoo.org/wiki/Handbook:AMD64
```

To go back to the original terminal, press `Alt+F1`.

# Optional: Starting the SSH daemon

To allow other users to access the system during the installation (perhaps to support during an installation, or even do it remotely), a user account needs to be created (as was documented earlier on) and the SSH daemon needs to be started.

To fire up the SSH daemon on an OpenRC init, execute the following command:

```
livecd ~ # rc-service sshd start
```

# Connecting wireless internet

https://wiki.gentoo.org/wiki/Handbook:AMD64/Networking/Wireless

Configure wpa_supplicant by creating the file `/etc/wpa_supplicant/wpa_supplicant.conf` with the sample content below:

```
ctrl_interface=/var/run/wpa_supplicant

network={
  ssid="<network_name>"
  psk="<network_password>"
  priority=1
}
```

Connect to internet with the following command:

```
livecd ~ # nohup wpa_supplicant -Dnl80211 -iwlp5s0 -c/etc/wpa_supplicant.conf &
```

Test the internet:

```
livecd ~ # ping -c4 gentoo.org
```

Connect from a different machine via **ssh** command:

```
nelson ~ $ ssh root@<ip>
```

# Default partitioning scheme

Throughout the remainder of the handbook, we will discuss and explain two cases: 1) GPT partition table and UEFI boot, and 2) MBR partition table and legacy BIOS boot. While it is possible to mix and match, that goes beyond the scope of this manual. As already stated above, installations on modern hardware should use GPT partition table and UEFI boot; as an exception from this rule, MBR and BIOS boot is still frequently used in virtualized (cloud) environments.

The following partitioning scheme will be used as a simple example layout:

| **Partition** | **Filesystem** | **Size** | **Description** |
| --- | --- | --- | --- |
| /dev/sda1 | fat32 (UEFI) or ext2 (BIOS) | 256 Mb | Boot/EFI system partition |
| /dev/sda2 | (swap) | RAM size * 2 | Swap Partition |
| /dev/sda3 | ext4 | Rest of the disk | Root Partition |

# Partitioning the disk with MBR for BIOS / legacy boot

Fire up **fdisk** against the disk (in our example, we use /dev/sda):

```
livecd ~ # fdisk /dev/sda

Welcome to fdisk (util-linux 2.36.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): 
```

Use the `p` key to display the disk's current partition configuration:

```
Command (m for help): p
Disk /dev/sda: 465.76 GiB, 500107862016 bytes, 976773168 sectors
Disk model: Samsung SSD 860 
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x2b8a5a98

Device     Boot    Start       End   Sectors   Size Id Type
/dev/sda1  *        2048    526335    524288   256M 83 Linux
/dev/sda2         526336  34080767  33554432    16G 82 Linux swap / Solaris
/dev/sda3       34080768 976773167 942692400 449.5G 83 Linux

Command (m for help):  
```

Use `d` and press `Enter` repeatedly to delete the old partitions (**make sure we have backed up the info previously**):

```
Command (m for help): d
Partition number (1-3, default 3): 

Partition 3 has been deleted.

Command (m for help): d
Partition number (1,2, default 2): 

Partition 2 has been deleted.

Command (m for help): d
Selected partition 1
Partition 1 has been deleted.
```

Use `p` again to see the changes:

```
Command (m for help): p
Disk /dev/sda: 465.76 GiB, 500107862016 bytes, 976773168 sectors
Disk model: Samsung SSD 860 
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x2b8a5a98
```

Now we're ready to create the partitions.

Use `n` and create the **Boot Partition**:

```
Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): 

Using default response p.
Partition number (1-4, default 1): 
First sector (2048-976773167, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-976773167, default 976773167): +256M

Created a new partition 1 of type 'Linux' and of size 256 MiB.
Partition #1 contains a ext2 signature.

Do you want to remove the signature? [Y]es/[N]o: Y

The signature will be removed by a write command.
```

Use `n` and create the **Swap Partition**:

```
Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): 

Using default response p.
Partition number (2-4, default 2): 
First sector (526336-976773167, default 526336): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (526336-976773167, default 976773167): +8G

Created a new partition 2 of type 'Linux' and of size 8 GiB.
Partition #2 contains a swap signature.

Do you want to remove the signature? [Y]es/[N]o: Y

The signature will be removed by a write command.
```

Use `n` and create the **Root Partition**:

```
Command (m for help): n
Partition type
   p   primary (2 primary, 0 extended, 2 free)
   e   extended (container for logical partitions)
Select (default p): 

Using default response p.
Partition number (3,4, default 3): 
First sector (17303552-976773167, default 17303552): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (17303552-976773167, default 976773167): 

Created a new partition 3 of type 'Linux' and of size 457.5 GiB.
```

Use `t` and change the type of the **Swap Partition**:

```
Command (m for help): t 
Partition number (1-3, default 3): 2
Hex code or alias (type L to list all): L

00 Empty            24 NEC DOS          81 Minix / old Lin  bf Solaris        
01 FAT12            27 Hidden NTFS Win  82 Linux swap / So  c1 DRDOS/sec (FAT-
02 XENIX root       39 Plan 9           83 Linux            c4 DRDOS/sec (FAT-
03 XENIX usr        3c PartitionMagic   84 OS/2 hidden or   c6 DRDOS/sec (FAT-
04 FAT16 <32M       40 Venix 80286      85 Linux extended   c7 Syrinx         
05 Extended         41 PPC PReP Boot    86 NTFS volume set  da Non-FS data    
06 FAT16            42 SFS              87 NTFS volume set  db CP/M / CTOS / .
07 HPFS/NTFS/exFAT  4d QNX4.x           88 Linux plaintext  de Dell Utility   
08 AIX              4e QNX4.x 2nd part  8e Linux LVM        df BootIt         
09 AIX bootable     4f QNX4.x 3rd part  93 Amoeba           e1 DOS access     
0a OS/2 Boot Manag  50 OnTrack DM       94 Amoeba BBT       e3 DOS R/O        
0b W95 FAT32        51 OnTrack DM6 Aux  9f BSD/OS           e4 SpeedStor      
0c W95 FAT32 (LBA)  52 CP/M             a0 IBM Thinkpad hi  ea Linux extended 
0e W95 FAT16 (LBA)  53 OnTrack DM6 Aux  a5 FreeBSD          eb BeOS fs        
0f W95 Ext'd (LBA)  54 OnTrackDM6       a6 OpenBSD          ee GPT            
10 OPUS             55 EZ-Drive         a7 NeXTSTEP         ef EFI (FAT-12/16/
11 Hidden FAT12     56 Golden Bow       a8 Darwin UFS       f0 Linux/PA-RISC b
12 Compaq diagnost  5c Priam Edisk      a9 NetBSD           f1 SpeedStor      
14 Hidden FAT16 <3  61 SpeedStor        ab Darwin boot      f4 SpeedStor      
16 Hidden FAT16     63 GNU HURD or Sys  af HFS / HFS+       f2 DOS secondary  
17 Hidden HPFS/NTF  64 Novell Netware   b7 BSDI fs          fb VMware VMFS    
18 AST SmartSleep   65 Novell Netware   b8 BSDI swap        fc VMware VMKCORE 
1b Hidden W95 FAT3  70 DiskSecure Mult  bb Boot Wizard hid  fd Linux raid auto
1c Hidden W95 FAT3  75 PC/IX            bc Acronis FAT32 L  fe LANstep        
1e Hidden W95 FAT1  80 Old Minix        be Solaris boot     ff BBT            

Aliases:
   linux          - 83
   swap           - 82
   extended       - 05
   uefi           - EF
   raid           - FD
   lvm            - 8E
   linuxex        - 85
Hex code or alias (type L to list all): 82

Changed type of partition 'Linux' to 'Linux swap / Solaris'.
```

Use `p` once again to confirm the partitions' result:

```
Command (m for help): p
Disk /dev/sda: 465.76 GiB, 500107862016 bytes, 976773168 sectors
Disk model: Samsung SSD 860 
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x2b8a5a98

Device     Boot    Start       End   Sectors   Size Id Type
/dev/sda1           2048    526335    524288   256M 83 Linux
/dev/sda2         526336  17303551  16777216     8G 82 Linux swap / Solaris
/dev/sda3       17303552 976773167 959469616 457.5G 83 Linux
```

To save the partition layout and close it, use `w`:

```
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

livecd ~ # 
```

🗒️ **NOTE:**
When using ext2, ext3, or ext4 on a small partition (less than 8 GiB), then the file system must be created with the proper options to reserve enough inodes. This can be done using one of the following commands, respectively:
Then, create the filesystem for the partitions:

```
livecd ~ # mkfs.ext2 -T small -L "Boot" /dev/sda1
```

For UEFI systems, use the following command to create the boot partition:

```
livecd ~ # mkfs.fat -F 32 -n efi-boot /dev/sda1
```

Create and mount the **Swap** partition:

```
livecd ~ # mkswap -L "Swap Linux" /dev/sda2
livecd ~ # swapon /dev/sda2
```

Format the **Root** partition:

```
livecd ~ # mkfs.ext4 -L "Gentoo" /dev/sda3
```

Mount the **Root** and **Boot** partitions:

```
livecd ~ # mount /dev/sda3 /mnt/gentoo
livecd ~ # mkdir /mnt/gentoo/boot && mount /dev/sda1 /mnt/gentoo/boot
```

On UEFI systems, mount the EFI partition at the following point instead:

```
livecd ~ # mkdir -p /mnt/gentoo/boot/efi
livecd ~ # mount /dev/sda1 /mnt/gentoo/boot/efi
```

🗒️ **NOTE:**
If /tmp/ needs to reside on a separate partition, be sure to change its permissions after mounting:

```
livecd ~ # chmod 1777 /mnt/gentoo/tmp
```

This also holds for /var/tmp.

# Installing a stage tarball

**Setting the date and time**
Verify the current date and time by running the date command:

```
livecd ~ # date
Thu May 20 22:49:03 UTC 2021
```

If the date/time displayed is wrong, update it using one of the methods below:

**Automatic**

```
livecd ~ # ntpd -q -g
```

**Manual**
The date command can also be used to perform a manual set on the system clock. Use the MMDDhhmmYYYY syntax (Month, Day, hour, minute and Year).

UTC time is recommended for all Linux systems. Later on during the installation a timezone will be defined. This will modify the display of the clock to local time.

For instance, to set the date to May 20, 17:54 in the year 2021:

```
livecd ~ # date 052017542021
Thu May 20 17:54:00 UTC 2021
```

**Downloading the stage tarball**
Go to the Gentoo mount point where the root file system is mounted (most likely /mnt/gentoo):

```
livecd ~ # cd /mnt/gentoo
```

Use a Web Browser, and from the [Download Section](https://www.gentoo.org/downloads/#other-arches), simply select the appropriate tab, right click the link to the stage file, then Copy Link to copy the link to the clipboard, then paste the link to the **wget** utility on the command-line to download the stage systemd tarball:

```
livecd ~ # wget <PASTED_STAGE_URL>
```

**Unpacking the stage tarball**
Now unpack the downloaded stage onto the system. We use **tar** to proceed:

```
livecd /mnt/gentoo # tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
```

Make sure that the same options (`xpf` and `--xattrs-include='*.*'`) are used. The `x` stands for extract, the `p` for preserve permissions and the `f` to denote that we want to extract a file (not standard input). `--xattrs-include='*.*'` is to include preservation of the the extended attributes in all namespaces stored in the archive. Finally, `--numeric-owner` is used to ensure that the user and group IDs of the files being extracted from the tarball will remain the same as Gentoo's release engineering team intended (even if adventurous users are not using official Gentoo installation media).

# Configuring compile options

To optimize Gentoo, it is possible to set a couple of variables which impacts the behavior of Portage, Gentoo's officially supported package manager. All those variables can be set as environment variables (using **export**) but that isn't permanent. To keep the settings, Portage reads in the /etc/portage/make.conf file, a configuration file for Portage.

Fire up an editor (in this guide we use **vi**) to alter the optimization variables we will discuss hereafter.

```
livecd /mnt/gentoo # nano -w /mnt/gentoo/etc/portage/make.conf
```

Edit the file according with the pc requirements:

```
COMMON_FLAGS="-march=native -O2 -pipe"
```

💡 **Tip:**
Although the [GCC optimization](https://wiki.gentoo.org/wiki/GCC_optimization) article has more information on how the various compilation options can affect a system, the [Safe CFLAGS](https://wiki.gentoo.org/wiki/Safe_CFLAGS) article may be a more practical place for beginners to start optimizing their systems.

**MAKEOPTS**
The MAKEOPTS variable defines how many parallel compilations should occur when installing a package. A good choice is the number of CPUs (or CPU cores) in the system plus one, but this guideline isn't always perfect.

Using a large number of jobs can significantly impact memory consumption. A good recommendation is to have at least 2 GiB of RAM for every job specified (so, e.g. -j6 requires at least 12 GiB). To avoid running out of memory, lower the number of jobs to fit the available memory.

CODE Example MAKEOPTS declaration in make.conf

```
MAKEOPTS="-j2"
```

**Ready, set, go!**
Update the /mnt/gentoo/etc/portage/make.conf file to match personal preference and save.

# Chrooting

**Selecting mirrors**
In order to download source code quickly it is recommended to select a fast mirror. Portage will look in the make.conf file for the GENTOO_MIRRORS variable and use the mirrors listed therein. It is possible to surf to the Gentoo mirror list and search for a mirror (or mirrors) that is close to the system's physical location (as those are most frequently the fastest ones). However, we provide a nice tool called **mirrorselect** which provides users with a nice interface to select the mirrors needed. Just navigate to the mirrors of choice and press `Spacebar` to select one or more mirrors.

```
livecd /mnt/gentoo # mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf
```

**Gentoo ebuild repository**
A second important step in selecting mirrors is to configure the Gentoo ebuild repository via the /etc/portage/repos.conf/gentoo.conf file. This file contains the sync information needed to update the package repository (the collection of ebuilds and related files containing all the information Portage needs to download and install software packages).

Configuring the repository can be done in a few simple steps. First, if it does not exist, create the [repos.conf](https://wiki.gentoo.org/wiki//etc/portage/repos.conf) directory:

```
livecd /mnt/gentoo # mkdir --parents /mnt/gentoo/etc/portage/repos.conf
```

Next, copy the Gentoo repository configuration file provided by Portage to the (newly created) `repos.conf` directory:

```
livecd /mnt/gentoo # cp /usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
```

💡 **Tip:**
For those interested, the official specification for Portage's plug-in sync API can be found in the Portage project's [Sync article](https://wiki.gentoo.org/wiki/Project:Portage/Sync).

**Copy DNS info**
One thing still remains to be done before entering the new environment and that is copying over the DNS information in /etc/resolv.conf. This needs to be done to ensure that networking still works even after entering the new environment. /etc/resolv.conf contains the name servers for the network.

To copy this information, it is recommended to pass the `--dereference` option to the **cp** command. This ensures that, if /etc/resolv.conf is a symbolic link, that the link's target file is copied instead of the symbolic link itself. Otherwise in the new environment the symbolic link would point to a non-existing file (as the link's target is most likely not available inside the new environment).

```
livecd /mnt/gentoo # cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
```

**Mounting the necessary filesystems**
The filesystems that need to be made available are:

- /proc/ which is a pseudo-filesystem (it looks like regular files, but is actually generated on-the-fly) from which the Linux kernel exposes information to the environment
- /sys/ which is a pseudo-filesystem, like /proc/ which it was once meant to replace, and is more structured than /proc/
- /dev/ is a regular file system, partially managed by the Linux device manager (usually **udev**), which contains all device files

The /proc/ location will be mounted on /mnt/gentoo/proc/ whereas the other two are bind-mounted. The latter means that, for instance, /mnt/gentoo/sys/ will actually be /sys/ (it is just a second entry point to the same filesystem) whereas /mnt/gentoo/proc/ is a new mount (instance so to speak) of the filesystem.

```
livecd /mnt/gentoo # cd
```

```
livecd ~ # mount --types proc /proc /mnt/gentoo/proc
livecd ~ # mount --rbind /sys /mnt/gentoo/sys
livecd ~ # mount --make-rslave /mnt/gentoo/sys
livecd ~ # mount --rbind /dev /mnt/gentoo/dev
livecd ~ # mount --make-rslave /mnt/gentoo/dev
```

🗒️ **NOTE:**
The --make-rslave operations are needed for systemd support later in the installation.

**Entering the new environment**

This chrooting is done in three steps:

1.  List itemThe root location is changed from / (on the installation medium) to /mnt/gentoo/ (on the partitions) using chroot
2.  Some settings (those in /etc/profile) are reloaded in memory using the **source** command
3.  The primary prompt is changed to help us remember that this session is inside a chroot environment.

```
livecd ~ # chroot /mnt/gentoo /bin/bash
```

```
livecd ~ # source /etc/profile
```

```
livecd ~ # export PS1="(chroot) ${PS1}"
```

From this point, all actions performed are immediately on the new Gentoo Linux environment. Of course it is far from finished, which is why the installation still has some sections left!

💡 **Tip:**
If the Gentoo installation is interrupted anywhere after this point, it should be possible to 'resume' the installation at this step. There is no need to repartition the disks again! Simply [mount the root partition](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Disks#Mounting_the_root_partition) and run the steps above starting with [copying the DNS info](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Base#Copy_DNS_info) to re-enter the working environment. This is also useful for fixing bootloader issues. More information can be found in the [chroot](https://wiki.gentoo.org/wiki/Chroot) article.

# Installing a Gentoo ebuild repository snapshot from the web

Next step is to install a snapshot of the Gentoo ebuild repository. This snapshot contains a collection of files that informs Portage about available software titles (for installation), which profiles the system administrator can select, package or profile specific news items, etc.

The use of **emerge-webrsync** is recommended for those who are behind restrictive firewalls (it uses HTTP/FTP protocols for downloading the snapshot) and saves network bandwidth. Readers who have no network or bandwidth restrictions can happily skip down to the next section.

This will fetch the latest snapshot (which is released on a daily basis) from one of Gentoo's mirrors and install it onto the system:

```
(chroot) livecd / # emerge-webrsync
```

🗒️ **NOTE:**
During this operation, **emerge-webrsync** might complain about a missing /var/db/repos/gentoo/ location. This is to be expected and nothing to worry about - the tool will create the location.

From this point onward, Portage might mention that certain updates are recommended to be executed. This is because system packages installed through the stage file might have newer versions available; Portage is now aware of new packages because of the repository snapshot. Package updates can be safely ignored for now; updates can be delayed until after the Gentoo installation has finished.

**Optional: Updating the Gentoo ebuild repository**

It is possible to update the Gentoo ebuild repository to the latest version. The previous **emerge-webrsync** command will have installed a very recent snapshot (usually recent up to 24h) so this step is definitely optional.

Suppose there is a need for the last package updates (up to 1 hour), then use **emerge --sync**. This command will use the rsync protocol to update the Gentoo ebuild repository (which was fetched earlier on through **emerge-webrsync**) to the latest state.

```
(chroot) livecd / # emerge --sync
```

**Utilities**
Install `app-portage/eix` to search and query ebuilds. Also, `sys-apps/mlocate` to index and quickly search for files, and `app-editors/vim` for later usage:

```
(chroot) livecd / # emerge -a app-portage/eix sys-apps/mlocate app-editors/vim
```

# Vim Text Editor

```
(chroot) livecd / # eselect editor list
Available targets for the EDITOR variable:
  [1]   nano
  [2]   ex
  [3]   vi
  [ ]   (free form)
```

Use the following to select `Vi` editor as default:

```
(chroot) livecd ~ # eselect editor set 3
Setting EDITOR to vi ...
Run ". /etc/profile" to update the variable in your shell.
```

Then, update the variables:

```
(chroot) livecd ~ # . /etc/profile && export PS1="(chroot) ${PS1}"
```

# Tuning the new Gentoo

## MAKEOPTS

MAKEOPTS is a variable that defines how many parallel make jobs can be launched from Portage. It can be set in the [/etc/portage/make.conf](https://wiki.gentoo.org/wiki//etc/portage/make.conf#MAKEOPTS) configuration file.

On a system with one Intel Core i7 CPU the following command shows the numbering of the available logical CPUs.

```
root #lscpu
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit
Byte Order:          Little Endian
CPU(s):              8
On-line CPU(s) list: 0-8
Thread(s) per core:  2
Core(s) per socket:  4
Socket(s):           1
```

In this example 8 logical CPUs are available, 4 physical cores each with 2 threads, hence one could set the MAKEOPTS variable to:

```
FILE /etc/portage/make.conf
MAKEOPTS="-j8"
```

Use the following command to check how many cores the processor have (Ref.: [Parallel Builds With Gentoo's Emerge](https://www.preney.ca/paul/archives/341)):

```
gentoo-linux ~ # grep '^processor' /proc/cpuinfo | sort -u | wc -l
```

# Safe CFLAGS

Ref.: https://wiki.gentoo.org/wiki/Safe_CFLAGS

First. of all, confirm the processor's model with the following command:

```
gentoo-linux ~ # grep -m1 -A3 "vendor_id" /proc/cpuinfo
```

Identify the correct processor specifications with the following tools:

```
gentoo-linux ~ # emerge -a app-portage/cpuid2cpuflags app-misc/resolve-march-native
```

Run the tools to identify the correct details, then, edit the values from `COMMON_FLAGS` and `CPU_FLAGS_X86` on the `/etc/portage/make.conf` file.

# qtwebengine package

🗒️ **NOTE:**
The **qtwebengine** package is not required from me and takes more than a day to compile. Therefore, I don't include it on my installation.

Use the following commands to skip the **qtwebengine** and their dependencies requirements:

```
(chroot) livecd / # echo "dev-qt/qtwebengine" >> /etc/portage/package.mask/qtwebengine
```

```
(chroot) livecd / # echo "kde-misc/kio-gdrive" >> /etc/portage/package.mask/kio-gdrive
```

```
(chroot) livecd / # echo "kde-apps/kaccounts-providers" >> /etc/portage/package.mask/kaccounts-providers
```

```
(chroot) livecd / # echo "net-libs/signon-ui" >> /etc/portage/package.mask/signon-ui
```

The package `kde-misc/kio-gdrive` is also a dependency of the `kde-apps/kdenetwork-meta`, and this second package is required for the installation. The workaround is to create a `localrepo` with the package, and from this local ebuild, remove the `kde-misc/kio-gdrive` entry.

Use the following commands to create the `localrepo`:

```
(chroot) livecd / # mkdir -p /var/db/repos/localrepo/kde-apps/kdenetwork-meta
```

```
(chroot) livecd / # cd /var/db/repos/localrepo/kde-apps/kdenetwork-meta
```

```
(chroot) livecd /var/db/repos/localrepo/kde-apps/kdenetwork-meta # cp /var/db/repos/gentoo/kde-apps/kdenetwork-meta/kdenetwork-meta-21.04.1.ebuild .
```

```
(chroot) livecd /var/db/repos/localrepo/kde-apps/kdenetwork-meta # mkdir -p /var/db/repos/localrepo/metadata
```

```
(chroot) livecd /var/db/repos/localrepo/kde-apps/kdenetwork-meta # echo "masters = gentoo" >> /var/db/repos/localrepo/metadata/layout.conf
```

Create the file `/etc/portage/repos.conf/localrepo.conf` with the following content:

```
[localrepo]
location = /var/db/repos/localrepo
auto-sync = no
```

Open the file **kdenetwork-meta-21.04.1.ebuild** and remove the `kde-misc/kio-gdrive` entry `>=kde-misc/kio-gdrive-${PV}:${SLOT}`.

```
(chroot) livecd /var/db/repos/localrepo/kde-apps/kdenetwork-meta # nano -w kdenetwork-meta-21.04.1.ebuild
```

Then, save and close the file.

Mask the **kdenetwork-meta** from gentoo repository to use the one that is in the **localrepo** instead:

```
(chroot) livecd /var/db/repos/localrepo/kde-apps/kdenetwork-meta # echo "kde-apps/kdenetwork-meta::gentoo" >> /etc/portage/package.mask/kdenetwork-meta
```

Run the following command to create the Manifest for the local ebuild:

```
(chroot) livecd /var/db/repos/localrepo/kde-apps/kdenetwork-meta # ebuild kdenetwork-meta-21.04.1.ebuild manifest
>>> Creating Manifest for /var/db/repos/localrepo/kde-apps/kdenetwork-meta
(chroot) livecd /var/db/repos/localrepo/kde-apps/kdenetwork-meta # 
```

Use the eix and the mlocate, and update their data files:

```
(chroot) livecd /var/db/repos/localrepo/kde-apps/kdenetwork-meta # eix-update && updatedb
Reading Portage settings...
Building database (/var/cache/eix/portage.eix)...
[0] "gentoo" /var/db/repos/gentoo (cache: metadata-md5-or-flat)
     Reading category 167|167 (100) Finished             
[1] "" /var/db/repos/localrepo (cache: parse|ebuild*#metadata-md5#metadata-flat#assign)
     Reading category 167|167 (100) Finished    
Applying masks...
Calculating hash tables...
Writing database file /var/cache/eix/portage.eix...
Database contains 19338 packages in 167 categories
```

Return to the root path and create the repo_name:

```
(chroot) livecd /var/db/repos/localrepo/kde-apps/kdenetwork-meta # cd
```

```
(chroot) livecd / #
```

```
(chroot) livecd / # mkdir /var/db/repos/localrepo/profiles
```

```
(chroot) livecd / # echo "localrepo" >> /var/db/repos/localrepo/profiles/repo_name
```

# Configuring Portage

More `/etc/portage/make.conf` adjustments below:

```
## Initial KDE-Plasma profile installation with systemd
USE="-gtk -gnome qt4 qt5 kde dvd alsa cdr user-session sqlite icu obex bluetooth -handbook -webengine -debug -test networkmanager iwd -wpa_supplicant X"

ACCEPT_LICENSE="* -@EULA"

ACCEPT_KEYWORDS="~amd64"

## (For mouse, keyboard, and Synaptics touchpad support)
INPUT_DEVICES="libinput synaptics"

## (For NVIDIA cards)
VIDEO_CARDS="nouveau"

## (For Samsung M2070 Series Scanner support)
SANE_BACKENDS="xerox_mfp"
```

**Reading news items**
When the Gentoo ebuild repository is synchronized, Portage may output informational messages similar to the following:

```
* IMPORTANT: 2 news items need reading for repository 'gentoo'.
* Use eselect news to read news items.
```

News items were created to provide a communication medium to push critical messages to users via the Gentoo ebuild repository. To manage them, use **eselect news**. The **eselect** application is a Gentoo-specific utility that allows for a common management interface for system administration. In this case, **eselect** is asked to use its `news` module.

For the `news` module, three operations are most used:

- With `list` an overview of the available news items is displayed.
- With `read` the news items can be read.
- With `purge` news items can be removed once they have been read and will not be reread anymore.

```
(chroot) livecd / # eselect news list
News items:
  [1]   N  2016-06-19  L10N USE_EXPAND variable replacing LINGUAS
  [2]   N  2018-08-07  Migration required for OpenSSH with LDAP
  [3]   N  2019-05-23  Change of ACCEPT_LICENSE default
  [4]   N  2020-06-23  sys-libs/pam-1.4.0 upgrade
  [5]   N  2021-01-30  Python preference to follow PYTHON_TARGETS
  [6]   N  2021-05-05  Python 3.9 to become the default on 2021-06-01
```

Use the following command to read a single note:

```
(chroot) livecd / # eselect news read 5
```

According with the news 5, install the following package:

```
(chroot) livecd / # emerge -n app-eselect/eselect-python
```

According with the news 6, do the following commands:

```
(chroot) livecd / # echo "*/* PYTHON_TARGETS: -* python3_9" >> /etc/portage/package.use/python
```

```
(chroot) livecd / # echo "*/* PYTHON_SINGLE_TARGET: -* python3_9" >> /etc/portage/package.use/python
```

```
(chroot) livecd ~ # cat /etc/portage/package.use/python 
*/* PYTHON_TARGETS: -* python3_9
*/* PYTHON_SINGLE_TARGET: -* python3_9
```

To read all the news, use the following command:

```
(chroot) livecd ~ # eselect news read
```

**Choosing the right profile**

A profile is a building block for any Gentoo system. Not only does it specify default values for USE, CFLAGS, and other important variables, it also locks the system to a certain range of package versions. These settings are all maintained by Gentoo's Portage developers.

You can see what profile the system is currently using with **eselect**, now using the `profile` module:

```
(chroot) livecd ~ # eselect profile list
Available profile symlink targets:
  [1]   default/linux/amd64/17.1 (stable)
  [2]   default/linux/amd64/17.1/selinux (stable)
  [3]   default/linux/amd64/17.1/hardened (stable)
  [4]   default/linux/amd64/17.1/hardened/selinux (stable)
  [5]   default/linux/amd64/17.1/desktop (stable)
  [6]   default/linux/amd64/17.1/desktop/gnome (stable)
  [7]   default/linux/amd64/17.1/desktop/gnome/systemd (stable)
  [8]   default/linux/amd64/17.1/desktop/plasma (stable)
  [9]   default/linux/amd64/17.1/desktop/plasma/systemd (stable)
```

🗒️ **NOTE:**
If you are using Systemd, please make sure the profile name contains systemd.

To select the `desktop/plasma/systemd` profile, use the following command:

```
(chroot) livecd ~ # eselect profile set 9
```

**Updating the @world set**

At this point, it is wise to update the system's [@world set](https://wiki.gentoo.org/wiki/World_set_%28Portage%29 "https://wiki.gentoo.org/wiki/World_set_(Portage)") so that a base can be established.

This following step is necessary so the system can apply any updates or USE flag changes which have appeared since the stage3 was built and from any profile selection:

```
(chroot) livecd ~ # emerge -avuDN @world
```

🗒️ **NOTE:**
During the process, we could see that the package called `dev-lang/rust` normally takes several hours to compile, so I interrupt the prompt with `ctrl + c`, then I install the one that is called `dev-lang/rust-bin` instead.

**Configuring the USE variable**

Most distributions compile their packages with support for as much as possible, increasing the size of the programs and startup time, not to mention an enormous amount of dependencies. With Gentoo users can define what options a package should be compiled with. This is where USE comes into play.

The default USE settings are placed in the make.defaults files of the Gentoo profile used by the system. Gentoo uses a (complex) inheritance system for its profiles, which we will not dive into at this stage. The easiest way to check the currently active USE settings is to run **emerge --info** and select the line that starts with USE:

```
(chroot) livecd ~ # emerge --info | grep ^USE
USE="X a52 aac acl acpi activities alsa amd64 berkdb branding ..."
```

🗒️ **NOTE:**
The above example is truncated, the actual list of USE values is much, much larger.

A full description on the available USE flags can be found on the system in /var/db/repos/gentoo/profiles/use.desc.

```
(chroot) livecd ~ # less /var/db/repos/gentoo/profiles/use.desc
```

As an example we show a USE setting for a KDE-based system with DVD, ALSA, and CD recording support:

```
(chroot) livecd ~ # nano -w /etc/portage/make.conf
```

```
FILE /etc/portage/make.confEnabling flags for a KDE/Plasma-based system with DVD, ALSA, and CD recording support
USE="-gtk -gnome qt4 qt5 kde dvd alsa cdr"
```

When USE is defined in /etc/portage/make.conf it is added (or removed if the USE flag starts with the `-` sign) from that default list. Users who want to ignore any default USE settings and manage it completely themselves should start the USE definition in make.conf with `-*`:

```
FILE /etc/portage/make.confIgnoring default USE flags
USE="-* X acl alsa"
```

⚠ **Warning**
Although possible, setting `-*` (as seen in the example above) is discouraged since carefully chosen USE flag defaults may be configured for some packages to prevent conflicts and other errors.

# Optional: Using systemd as the init system

The remainder of the Gentoo Handbook focuses on [OpenRC](https://wiki.gentoo.org/wiki/OpenRC) (the traditional Gentoo init system) as the default init system. If systemd is desired, please consult the [systemd](https://wiki.gentoo.org/wiki/Systemd) article. It contains instructions equivalent to the instructions in the following sections of this Handbook. Specifically, it will walk the reader through various init system commands (systemctl) and systemd-specific services (such as **timedatectl**, **hostnamectl**, etc.) needed to establish a working systemd environment.

**Timezone - Systemd**
Select the timezone for the system. Look for the available timezones in /usr/share/zoneinfo/:

```
(chroot) livecd ~ # ls /usr/share/zoneinfo
```

🗒️ **NOTE:**
Suppose the timezone of choice is America/Bogota.

We use a slightly different approach here; we generate a symbolic link:

```
(chroot) livecd ~ # ln -sf ../usr/share/zoneinfo/America/Bogota /etc/localtime
```

Later, when systemd is running, we can configure the timezone and related settings with the **timedatectl** command.

# Configure locales

**Locale generation**

Most users will want to use only one or two locales on their system.

Locales specify not only the language that the user should use to interact with the system, but also the rules for sorting strings, displaying dates and times, etc. Locales are case sensitive and must be represented exactly as described. A full listing of available locales can be found in the /usr/share/i18n/SUPPORTED file.

Supported system locales must be defined in the /etc/locale.gen file.

```
(chroot) livecd ~ # vi /etc/locale.gen
```

The following locales are an example to get both English (United States) and Spanish (Colombia) with the accompanying character formats (like UTF-8).

```
FILE /etc/locale.genEnabling US and DE locales with the appropriate character formats
en_US ISO-8859-1
en_US.UTF-8 UTF-8
es_CO ISO-8859-1
es_CO.UTF-8 UTF-8
```

⚠ **Warning**
We strongly suggest adding at least one UTF-8 locale because many applications may require it to build properly.

The next step is to run the **locale-gen** command. This command generates all locales specified in the /etc/locale.gen file.

```
(chroot) livecd ~ # locale-gen
```

To verify that the selected locales are now available, run **locale -a**.

**Locale selection**

Once done, it is now time to set the system-wide locale settings. Again we use **eselect** for this, now with the `locale` module.

With **eselect locale list**, the available targets are displayed:

```
(chroot) livecd ~ # eselect locale list
/usr/bin/locale: Cannot set LC_CTYPE to default locale: No such file or directory
Available targets for the LANG variable:
  [1]   C
  [2]   C.utf8
  [3]   POSIX
  [4]   en_US
  [5]   en_US.iso88591
  [6]   en_US.utf8
  [7]   es_CO
  [8]   es_CO.iso88591
  [9]   es_CO.utf8
  [10]  C.UTF8 *
  [ ]   (free form)

```

With **eselect locale set (NUMBER)** the correct locale can be selected:

```
(chroot) livecd ~ # eselect locale set 6
```

Manually, this can still be accomplished through the /etc/env.d/02locale file and for Systemd the /etc/locale.conf file:

```
FILE /etc/locale.conf Manually setting system locale definitions
LANG="en_US.UTF-8"
LC_COLLATE="C.UTF-8"
```

Setting the locale will avoid warnings and errors during kernel and software compilations later in the installation.

Now reload the environment:

```
(chroot) livecd ~ # env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
```

A full [Localization guide](https://wiki.gentoo.org/wiki/Localization/Guide) to provide additional guidance through the locale selection process. Another interesting article is the [UTF-8 guide](https://wiki.gentoo.org/wiki/UTF-8) for very specific information to enable UTF-8 on the system.

# Configuring the Linux kernel

**Installing the sources**
The core around which all distributions are built is the Linux kernel. It is the layer between the user programs and the system hardware. Gentoo provides its users several possible kernel sources. A full listing with description is available at the [Kernel overview](https://wiki.gentoo.org/wiki/Kernel/Overview) page.

For amd64-based systems Gentoo recommends the [sys-kernel/gentoo-sources](https://packages.gentoo.org/packages/sys-kernel/gentoo-sources) package.

Choose an appropriate kernel source and install it using **emerge**:

```
(chroot) livecd ~ # emerge -a sys-kernel/gentoo-sources sys-kernel/linux-fimrware
```

Select the new kernel just installed:

```
(chroot) livecd ~ # eselect kernel set 1
```

This will install the Linux kernel sources in /usr/src/ in which a symbolic link called linux will be pointing to the installed kernel source:

```
(chroot) livecd ~ # ls -l /usr/src/
total 4
lrwxrwxrwx  1 root root   19 May 21 21:50 linux -> linux-5.12.5-gentoo
drwxr-xr-x 25 root root 4096 May 21 21:50 linux-5.12.5-gentoo
```

Now it is time to configure and compile the kernel sources. There are two approaches for this:

1.  The kernel is manually configured and built.
2.  A tool called **genkernel** is used to automatically build and install the Linux kernel.

We explain the manual configuration as the default choice here as it is the best way to optimize an environment.

🗒️ **NOTE:**
We are not covering the option with the **genkernel** tool in this guide, but if needed, we can follow the instructions at the next link https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Kernel.

**Manual configuration**
Manually configuring a kernel is often seen as the most difficult procedure a Linux user ever has to perform. Nothing is less true - after configuring a couple of kernels no-one even remembers that it was difficult ;)

However, one thing is true: it is vital to know the system when a kernel is configured manually. Most information can be gathered by emerging [sys-apps/pciutils](https://packages.gentoo.org/packages/sys-apps/pciutils) which contains the **lspci** command:

```
(chroot) livecd ~ # emerge -a sys-apps/pciutils
```

🗒️ **NOTE:**
Inside the chroot, it is safe to ignore any pcilib warnings (like pcilib: cannot open /sys/bus/pci/devices) that **lspci** might throw out.

Run the **lspci** command, and save the output in a file for later usage:

```
(chroot) livecd ~ # lspci > lspci.chroot-livecd.txt
```

Another source of system information is to run **lsmod** to see what kernel modules the installation CD uses as it might provide a nice hint on what to enable.

Run the **lsmod** command, and save the output in a file for later usage:

```
(chroot) livecd ~ # lsmod > lsmod.chroot-livecd.txt
```

Now go to the kernel source directory and execute **make menuconfig**. This will fire up menu-driven configuration screen.

```
(chroot) livecd ~ # cd /usr/src/linux
```

```
(chroot) livecd /usr/src/linux # make menuconfig
```

The Linux kernel configuration has many, many sections. Let's first list some options that must be activated (otherwise Gentoo will not function, or not function properly without additional tweaks). We also have a [Gentoo kernel configuration guide](https://wiki.gentoo.org/wiki/Kernel/Gentoo_Kernel_Configuration_Guide) on the Gentoo wiki that might help out further.

**Activating required options**

If you are using [sys-kernel/gentoo-sources](https://packages.gentoo.org/packages/sys-kernel/gentoo-sources), we strongly recommend you enable the Gentoo-specific configuration options. These ensure that a minimum of kernel features required for proper functioning is available:

```
KERNEL Enabling Gentoo-specific options
Gentoo Linux --->
  Generic Driver Options --->
    [*] Gentoo Linux support
    [*]   Linux dynamic and persistent device naming (userspace devfs) support
    [*]   Select options required by Portage features
        Support for init systems, system and service managers  --->
          [ ] OpenRC, runit and other script based systems and managers
          [*] systemd
```

Naturally your choice in the last two lines depends on your choice of init system (OpenRC vs. Systemd).

🗒️ **NOTE:**
Make sure that every driver that is vital to the booting of the system (such as SCSI controller, etc.) is compiled in the kernel and not as a module, otherwise the system will not be able to boot completely.

Next select the exact processor type. It is also recommended to enable MCE features (if available) so that users are able to be notified of any hardware problems. On some architectures (such as x86_64), these errors are not printed to **dmesg**, but to /dev/mcelog. This requires the [app-admin/mcelog](https://packages.gentoo.org/packages/app-admin/mcelog) package.

```
(chroot) livecd /usr/src/linux # emerge -a app-admin/mcelog
```

After installing the package, we will receive the following note:

```
*	CONFIG_X86_MCELOG_LEGACY:	is not set when it should be.
```

Look for the option in the kernel and set it.

```
(chroot) livecd /usr/src/linux # grep CONFIG_X86_MCELOG_LEGACY .config
# CONFIG_X86_MCELOG_LEGACY	is not set
(chroot) livecd /usr/src/linux # sed -i 's/# CONFIG_X86_MCELOG_LEGACY	is not set/CONFIG_X86_MCELOG_LEGACY=y/g' .config
```

Then, `make menuconfig` again.

Also select Maintain a devtmpfs file system to mount at /dev so that critical device files are already available early in the boot process (CONFIG\_DEVTMPFS and CONFIG\_DEVTMPFS_MOUNT):

```
KERNEL Enabling devtmpfs support
Device Drivers --->
  Generic Driver Options --->
    [*] Maintain a devtmpfs filesystem to mount at /dev
    [*]   Automount devtmpfs at /dev, after the kernel mounted the rootfs
```

Verify SCSI disk support has been activated (CONFIG\_BLK\_DEV_SD):

```
KERNEL Enabling SCSI disk support
Device Drivers --->
   SCSI device support  --->
      <*> SCSI disk support
```

Now go to File Systems and select support for the filesystems you use. Don't compile the file system that is used for the root filesystem as module, otherwise the Gentoo system will not be able to mount the partition. Also select Virtual memory and /proc file system. Select one or more of the following options as needed by the system (CONFIG\_EXT2\_FS, CONFIG\_EXT3\_FS, CONFIG\_EXT4\_FS, CONFIG\_MSDOS\_FS, CONFIG\_VFAT\_FS, CONFIG\_PROC\_FS, and CONFIG_TMPFS):

```
KERNEL Selecting necessary file systems
File systems --->
  <*> Second extended fs support
  <*> The Extended 3 (ext3) filesystem
  <*> The Extended 4 (ext4) filesystem
  <*> Reiserfs support
  <*> JFS filesystem support
  <*> XFS filesystem support
  <*> Btrfs filesystem support
  DOS/FAT/NT Filesystems  --->
      <*> MSDOS fs support
      <*> VFAT (Windows-95) fs support
      (437) Default codepage for FAT (NEW)
      (iso8859-1) Default iocharset for FAT (NEW)
      [ ]   Enable FAT UTF-8 option by default (NEW)
      <*> exFAT filesystem support
      (utf8) Default iocharset for exFAT (NEW)
      <*> NTFS file system support
      [ ]   NTFS debugging support (NEW)
      [*]   NTFS write support
 
Pseudo Filesystems --->
    [*] /proc file system support
    [*] Tmpfs virtual memory file system support (former shm fs)
```

Most systems also have multiple cores at their disposal, so it is important to activate Symmetric multi-processing support (CONFIG_SMP):

```
KERNEL Activating SMP support
Processor type and features  --->
  [*] Symmetric multi-processing support
```

🗒️ **NOTE:**
In multi-core systems, each core counts as one processor.

If USB input devices (like keyboard or mouse) or other USB devices will be used, do not forget to enable those as well (CONFIG\_HID\_GENERIC and CONFIG\_USB\_HID, CONFIG\_USB\_SUPPORT, CONFIG\_USB\_XHCI\_HCD, CONFIG\_USB\_EHCI\_HCD, CONFIG\_USB\_OHCI_HCD):

```
KERNEL Activating USB support for input devices
Device Drivers --->
  HID support  --->
    -*- HID bus support
    <*>   Generic HID driver
    [*]   Battery level reporting for HID devices
      USB HID support  --->
        <*> USB HID transport layer
  [*] USB support  --->
    <*>     xHCI HCD (USB 3.0) support
    <*>     EHCI HCD (USB 2.0) support
    <*>     OHCI HCD (USB 1.1) support
```

Architecture specific kernel configuration
Make sure to select IA32 Emulation if 32-bit programs should be supported (CONFIG\_IA32\_EMULATION). Gentoo installs a multilib system (mixed 32-bit/64-bit computing) by default, so unless a no-multilib profile is used, this option is required.

🗒️ **NOTE:**
In this case, I will use Intel, but for AMD Processor we will need to use `AMD MCE Features` option.

```
KERNEL Selecting processor types and features
Processor type and features  --->
   [*] Machine Check / overheating reporting 
   [*]   Intel MCE Features
   [ ]   AMD MCE Features
   Processor family (AMD-Opteron/Athlon64)  --->
      ( ) Opteron/Athlon64/Hammer/K8
      ( ) Intel P4 / older Netburst based Xeon
      ( ) Core 2/newer Xeon
      ( ) Intel Atom
      (*) Generic-x86-64
Executable file formats / Emulations  --->
   [*] IA32 Emulation
```

Enable GPT partition label support if that was used previously when partitioning the disk (CONFIG\_PARTITION\_ADVANCED and CONFIG\_EFI\_PARTITION):

```
KERNEL Enable support for GPT
-*- Enable the block layer --->
   Partition Types --->
      [*] Advanced partition selection
      [*] EFI GUID Partition support
```

Enable EFI stub support and EFI variables in the Linux kernel if UEFI is used to boot the system (CONFIG\_EFI, CONFIG\_EFI\_STUB, CONFIG\_EFI\_MIXED, and CONFIG\_EFI_VARS):

```
KERNEL Enable support for UEFI
Processor type and features  --->
    [*] EFI runtime service support 
    [*]   EFI stub support
    [*]     EFI mixed-mode support
 
Firmware Drivers  --->
    EFI (Extensible Firmware Interface) Support  --->
        <*> EFI Variable Support via sysfs
```

# [systemd](https://wiki.gentoo.org/wiki/Systemd) support

```
KERNEL Quick setup using gentoo-sources
Gentoo Linux --->
   Support for init systems, system and service managers --->
      [*] systemd
```

To configure the kernel options manually (which is the only option when not using sys-kernel/gentoo-sources), the following kernel configuration options are required or recommended:

```
KERNEL Mandatory options
General setup  --->
    [*] Control Group support --->
        [*]   Support for eBPF programs attached to cgroup
    [ ] Enable deprecated sysfs features to support old userspace tools
    [*] Configure standard kernel features (expert users)  --->
        [*] open by fhandle syscalls
        [*] Enable eventpoll support
        [*] Enable signalfd() system call
        [*] Enable timerfd() system call
    [*] Enable bpf() system call
[*] Networking support --->
Device Drivers  --->
    Generic Driver Options  --->
        [*] Maintain a devtmpfs filesystem to mount at /dev
File systems  --->
    [*] Inotify support for userspace
    Pseudo filesystems  --->
        [*] /proc file system support
        [*] sysfs file system support
```

```
KERNEL Recommended options
General setup  --->
    [*] Configure standard kernel features (expert users)  --->
    [*] Checkpoint/restore support
    [*] Namespaces support  --->
        [*] Network namespace
[*] Enable the block layer  --->
    [*] Block layer SG support v4
General architecture-dependent options  --->
    [*] Enable seccomp to safely compute untrusted bytecode
Networking support --->
    Networking options --->
        <*> The IPv6 protocol
Device Drivers  --->
    Generic Driver Options  --->
        [*] Support for uevent helper
        ()  path to uevent helper
        [ ] Fallback user-helper invocation for firmware loading
Firmware Drivers  --->
    [*] Export DMI identification via sysfs to userspace
File systems --->
    <*> Kernel automounter version 4 support (also supports v3)
    Pseudo filesystems --->
        [*] Tmpfs virtual memory file system support (former shm fs)
        [*]   Tmpfs POSIX Access Control Lists
        [*]   Tmpfs extended attributes
```

For an UEFI system also enable the following:

```
KERNEL UEFI support
[*] Enable the block layer  --->
    Partition Types  --->
        [*] Advanced partition selection
        [*]   EFI GUID Partition support
Processor type and features  --->
    [*] EFI runtime service support
Firmware Drivers  --->
        EFI (Extensible Firmware Interface) Support -->
            <*> EFI Variable Support via sysfs
```

🗒️ **NOTE:**
Don't forget to include support in the kernel for the network (Ethernet or wireless) cards.

Start with the instructions for [Wifi](https://wiki.gentoo.org/wiki/Wifi), including the required device drivers and IEEE802.11 support.

Additional [firmware](https://wiki.gentoo.org/wiki/Linux_firmware) for the individual device is needed as listed in [this table](https://wireless.wiki.kernel.org/en/users/drivers/iwlwifi#firmware). It is available in [sys-kernel/linux-firmware](https://packages.gentoo.org/packages/sys-kernel/linux-firmware). In case it's not in linux-firmware it might be found in device-specific [sys-firmware/iwlxxxx-*ucode](https://packages.gentoo.org/packages/search?q=sys-firmware%2Fiwl) packages.

For the `Firmware for Intel (R) Dual Band Wireless-AC 3160` device, is required to install the firmware. Run the following command to install it:

```
(chroot) livecd /usr/src/linux # emerge -a sys-kernel/linux-fimrware
```

**IEEE 802.11**

Activate at least [cfg80211](https://wireless.wiki.kernel.org/en/developers/documentation/cfg80211) and [mac80211](https://wireless.wiki.kernel.org/en/developers/documentation/mac80211).

```
KERNEL linux-4.19
[*] Networking support  --->
    [*] Wireless  --->
        <M>   cfg80211 - wireless configuration API
        [ ]     nl80211 testmode command
        [ ]     enable developer warnings
        [ ]     cfg80211 certification onus
        [*]     enable powersave by default
        [ ]     cfg80211 DebugFS entries
        [ ]     support CRDA
        [ ]     cfg80211 wireless extensions compatibility
        <M>   Generic IEEE 802.11 Networking Stack (mac80211)
        [*]   Minstrel
        [*]     Minstrel 802.11n support
        [ ]       Minstrel 802.11ac support
              Default rate control algorithm (Minstrel)  --->
        [ ]   Enable mac80211 mesh networking (pre-802.11s) support
        -*-   Enable LED triggers
        [ ]   Export mac80211 internals in DebugFS
        [ ]   Trace all mac80211 debug messages
        [ ]   Select mac80211 debugging features  ----
```

Minstrel and its 802.11n support is a [rate control algorithm](https://wireless.wiki.kernel.org/en/developers/Documentation/mac80211/RateControl/minstrel). Some wireless drivers might require it enabled.

**Important**

In case the **wireless configuration API** (CONFIG_CFG80211) is built into the kernel (`<*>`) instead as a module (`<M>`), the driver won't be able to load regulatory.db from /lib/firmware resulting in broken regulatory domain support. Please set CONFIG\_CFG80211=m or add regulatory.db and regulatory.db.p7s to CONFIG\_EXTRA_FIRMWARE.

**WEXT**

The "cfg80211 wireless [extensions compatibility](https://wireless.wiki.kernel.org/en/developers/documentation/wireless-extensions)" option aka WEXT will support old [**wireless-tools**](https://wiki.gentoo.org/wiki/Handbook:Parts/Networking/Wireless#Wireless_tools) and [**iwconfig**](https://wireless.wiki.kernel.org/en/users/documentation/iw/replace-iwconfig).

```
KERNEL
[*] Networking support  --->
    [*] Wireless  --->
        [*]     cfg80211 wireless extensions compatibility

```

**Device drivers**

Next the right set of corresponding kernel options need to be enabled, based on the drivers and hardware detected previously. The [recommendation](https://forums.gentoo.org/viewtopic-p-7640140.html#7640140) is to build [drivers as modules](https://wiki.gentoo.org/wiki/Kernel_Modules#Compile-in-kernel_modules_vs_Loadable_kernel_modules_.28LKMs.29). Also be sure to enable AES cipher support in the kernel if the wireless network uses WPA or WPA2 encryption.

```
KERNEL
Device Drivers  --->
    [*] Network device support  --->
        [*] Wireless LAN  --->
 
            Select the driver for your Wifi network device, e.g.:
            <M> Broadcom 43xx wireless support (mac80211 stack) (b43)
            [M]    Support for 802.11n (N-PHY) devices
            [M]    Support for low-power (LP-PHY) devices
            [M]    Support for HT-PHY (high throughput) devices
            <M> Intel Wireless WiFi Next Gen AGN - Wireless-N/Advanced-N/Ultimate-N (iwlwifi)
            <M>    Intel Wireless WiFi DVM Firmware support                             
            <M>    Intel Wireless WiFi MVM Firmware support
            <M> Intel Wireless WiFi 4965AGN (iwl4965)
            <M> Intel PRO/Wireless 3945ABG/BG Network Connection (iwl3945)
            <M> Ralink driver support  --->
                <M>   Ralink rt27xx/rt28xx/rt30xx (USB) support (rt2800usb)
 
-*- Cryptographic API --->
    -*- AES cipher algorithms
    -*- AES cipher algorithms (x86_64)
    <*> AES cipher algorithms (AES-NI)
```

**Important**

In case the driver is built into the kernel (`<*>`) instead of as a module (`<M>`), then the firmware needs to be built [into the kernel](https://wiki.gentoo.org/wiki/Kernel_Modules#Compile-in-kernel_modules_vs_Loadable_kernel_modules_.28LKMs.29) as well.
Do not forget to [rebuild the kernel](https://wiki.gentoo.org/wiki/Kernel/Rebuild) after changing its configuration.

**LED support**

To enable LED triggers for different packet receive/transmit events, compile the kernel with the following options:

```
KERNEL
Device Drivers  --->
    [*] LED Support  --->
        <*>   LED Class Support
 
[*] Networking support  --->
    [*] Wireless  --->
        [*] Enable LED triggers
```

Iwd requires the Linux kernel to have quite some options to be enabled. For systems running on a [AMD64](https://wiki.gentoo.org/wiki/AMD64) architecture, or CPUs that support SSSE3 or X86_AES instructions some hardware acceleration can be achieved. [Cpuid2cpuflags](https://wiki.gentoo.org/wiki/CPU_FLAGS_X86) can be used to check for support.

Specifics for [iwd](https://wiki.gentoo.org/wiki/Iwd) are described below:

```
KERNEL
Security options  --->
    [*] Enable access key retention support
    [*] Diffie-Hellman operations on retained keys
Networking support  --->
    [*] Wireless  --->
        <M> cfg80211 - wireless configuration API
Cryptographic API  --->
    *** Public-key cryptography ***
    [*] RSA algorithm
    [*] Diffie-Hellman algorithm
    *** Block modes ***
    [*] ECB support
    *** Hash modes ***
    [*] HMAC support
    *** Digest ***
    [*] MD4 digest algorithm
    [*] MD5 digest algorithm
    [*] SHA1 digest algorithm
    [*] SHA1 digest algorithm (SSSE3/AVX/AVX2/SHA-NI)   // AMD64 and SSSE3
    [*] SHA224 and SHA256 digest algorithm
    [*] SHA256 digest algorithm (SSSE3/AVX/AVX2/SHA-NI) // AMD64 and SSSE3
    [*] SHA384 and SHA512 digest algorithms
    [*] SHA512 digest algorithm (SSSE3/AVX/AVX2)        // AMD64 and SSSE3
    *** Ciphers **
    [*] AES cipher algorithms
    [*] AES cipher algorithms (x86_64)                  // AMD64
    [*] AES cipher algorithms (AES-NI)                  // X86_AES
    [*] ARC4 cipher algorithm
    [*] DES and Triple DES EDE cipher algorithms
    [*] Triple DES EDE cipher algorithm (x86-64)        // AMD64
    *** Random Number Generation ***
    [*] User-space interface for hash algorithms 
    [*] User-space interface for symmetric key cipher algorithms
    [*] Asymmetric (public-key cryptographic) key type  --->
        [*] Asymmetric public-key crypto algorithm subtype 
        [*] X.509 certificate parser
        [*] PKCS#7 message parser
        [*] PKCS#8 private key parser                   // linux kernel 4.20 or higher
```

## Xorg kernel requirements

Before you can install Xorg, you need to prepare your system for it. First, we'll set up the kernel to support input devices and video cards. Then we'll prepare [/etc/portage/make.conf](https://wiki.gentoo.org/wiki//etc/portage/make.conf) so that the right drivers and Xorg packages are built and installed.

The second variable is [INPUT_DEVICES](https://wiki.gentoo.org/wiki//etc/portage/make.conf#INPUT_DEVICES) and is used to determine which drivers are to be built for input devices.

[make.defaults](https://gitweb.gentoo.org/repo/gentoo.git/commit/?id=d3ac878318dd96a88190a13b5ac7572ec0c56380) has [Libinput](https://wiki.gentoo.org/wiki/Libinput) as the default input device driver.

To check what is presently set, run:

```
(chroot) livecd /usr/src/linux # portageq envvar INPUT_DEVICES
```

In case alternative input devices, such as a Synaptics touchpad for a laptop are needed, be sure to add them to INPUT_DEVICES the /etc/portage/make.conf file:

```
CODE Sample make.conf entries
## (For mouse, keyboard, and Synaptics touchpad support)
INPUT_DEVICES="libinput synaptics"
## (For NVIDIA cards)
VIDEO_CARDS="nouveau"
## (For AMD/ATI cards)
VIDEO_CARDS="radeon"
```

**Input driver support**

Support for Event interface (CONFIG\_INPUT\_EVDEV) needs to be activated by making a change to the kernel configuration. Read the Kernel Configuration Guide if you don't know how to setup your kernel.

```
KERNEL Enabling evdev in the kernel
Device Drivers --->
  Input device support --->
  <*>  Event interface
```

**Kernel modesetting**

Modern open source video drivers rely on kernel mode setting (KMS). KMS provides an improved graphical boot with less flickering, faster user switching, a built-in framebuffer console, seamless switching from the console to Xorg, and other features.

**Verify legacy framebuffer drivers have been disabled**

**Important**
KMS conflicts with legacy framebuffer drivers, which must remain disabled in the kernel configuration.
First prepare the kernel for KMS. This step regardless of which Xorg video driver will be used:

```
KERNEL Disable legacy framebuffer support and enable basic console FB support
Device Drivers --->
   Graphics support --->
      Frame Buffer Devices --->
         <*> Support for frame buffer devices --->
         ## (Disable all drivers, including VGA, Intel, NVIDIA, and ATI, except EFI-based Framebuffer Support, only if you are using UEFI)
 
      ## (Further down, enable basic console support. KMS uses this.)
      Console display driver support --->
         <*>  Framebuffer Console Support
```

**Intel**

For Intel cards see the [kernel section of the Intel article](https://wiki.gentoo.org/wiki/Intel#Kernel).

**NVIDIA**

For NVIDIA cards, two driver options are available. For a full open source system, an open source driver entitled [Nouveau](https://wiki.gentoo.org/wiki/Nouveau) is suggested. The second option is the closed source [NVIDIA driver](https://wiki.gentoo.org/wiki/NVIDIA), which is officially supported by NVIDIA. This article recommends the Nouveau driver, however be aware not all functionally for certain cards may be supported using the open source driver.

In addition to the kernel driver, certain cards require closed source firmware to be built-in to the Linux kernel. Depending on the selected driver, readers should visit each respective article to check to see if firmware (from the [sys-kernel/linux-firmware](https://packages.gentoo.org/packages/sys-kernel/linux-firmware) is necessary for their specific card.

```
KERNEL NVIDIA kernel
Device Drivers --->
   Graphics support --->
      <M/*>  Nouveau (NVIDIA) cards
```

AMD/ATI
For newer AMD/ATI cards ([RadeonHD 2000 and up](https://wiki.gentoo.org/wiki/ATI_FAQ)), emerge [sys-kernel/linux-firmware](https://packages.gentoo.org/packages/sys-kernel/linux-firmware) (the package includes firmware for radeon and amdgpu drivers). Once one of these packages has been installed, make the Radeon driver a module in the kernel or, optionally, configure the kernel as detailed in the [firmware section](https://wiki.gentoo.org/wiki/Radeon#Firmware) of the Radeon article or, for newer AMD graphics cards (GCN1.1+), the [firmware section](https://wiki.gentoo.org/wiki/AMDGPU#Firmware) of the AMDGPU article.

**Older cards:**

```
KERNEL AMD/ATI Radeon settings
## (Setup the kernel to use the radeon-ucode firmware, optional if "ATI Radeon" below is M)
Device Drivers --->
   Generic Driver Options --->
   [*]  Include in-kernel firmware blobs in kernel binary
  ## # ATI card specific, (see Radeon page for details which firmware files to include)
   (radeon/(CARD-MODEL).bin ...)
  ## # Specify the root directory
   (/lib/firmware/) External firmware blobs to build into the kernel binary
 
## (Enable Radeon KMS support)
Device Drivers --->
   Graphics support --->
   <M/*> Direct Rendering Manager (XFree86 4.1.0 and higher DRI support) --->
   <M/*>    ATI Radeon
   [*]      Enable modesetting on radeon by default
   [ ]      Enable userspace modesetting on radeon (DEPRECATED)
```

**Newer cards:**

```
KERNEL AMDGPU settings
## (Setup the kernel to use the amdgpu firmware, optional if "AMD GPU" below is M)
Device Drivers --->
   Generic Driver Options --->
   [*]  Include in-kernel firmware blobs in kernel binary
  ## # AMD card specific, (see AMDGPU page for details which firmware files to include)
   (amdgpu/(CARD-MODEL).bin ...)
  ## # Specify the root directory
   (/lib/firmware/) External firmware blobs to build into the kernel binary
 
## (Enable Radeon KMS support)
Device Drivers --->
   Graphics support --->
   <M/*> Direct Rendering Manager (XFree86 4.1.0 and higher DRI support) --->
   <M/*> AMD GPU
         [ /*] Enable amdgpu support for SI parts
         [ /*] Enable amdgpu support for CIK parts 
         [*]   Enable AMD powerplay component  
         ACP (Audio CoProcessor) Configuration  ---> 
             [*] Enable AMD Audio CoProcessor IP support (CONFIG_DRM_AMD_ACP)
         Display Engine Configuration  --->
             [*] AMD DC - Enable new display engine
             [ /*] DC support for Polaris and older ASICs
             [ /*] AMD FBC - Enable Frame Buffer Compression
             [ /*] DCN 1.0 Raven family
   <M/*> HSA kernel driver for AMD GPU devices
```

🗒️ **NOTE:**
Older Radeon cards (X1900 series and older) do not need extra firmware or any special firmware configuration. Direct Rendering Manager (DRM) and ATI Radeon modesetting driver are the only kernel settings necessary for correct operation.

🗒️ **NOTE:**
Linux kernel >= 3.9 does not have the Enable modesetting on radeon by default since it is already implied by default. Do not be alarmed if you find this option missing in new kernels.

🗒️ **NOTE:**
Linux kernel >= 4.15 does include Display Core (DC) which is required for AMDGPU to work. This newer driver was written for GCN5.0 Vega and DCN1.0 Raven Ridge (APU), but also adds additional functionality for older Radeon graphics cards starting with GCN1.1 Southern Islands and newer. It is planned to make this additional support for older Radeon cards the standard, so do not be alarmed if you find this option missing in newer kernels.

# Compiling and installing

With the configuration now done, it is time to compile and install the kernel. Exit the configuration and start the compilation process:

```
(chroot) livecd /usr/src/linux # make && make modules_install
```

🗒️ **NOTE:**
It is possible to enable parallel builds using make -jX with `X` being an integer number of parallel tasks that the build process is allowed to launch. This is similar to the instructions about /etc/portage/make.conf earlier, with the MAKEOPTS variable.
When the kernel has finished compiling, copy the kernel image to /boot/. This is handled by the make install command:

```
(chroot) livecd /usr/src/linux # make install
```

This will copy the kernel image into /boot/ together with the System.map file and the kernel configuration file.

🗒️ **NOTE:**
Once we have installed GRUB, everytime we make changes in the kernel configuration is must recommended to create the new entry for the boot in the GRUB menu. For that, use the following command:

```
(chroot) livecd /usr/src/linux # grub-mkconfig -o /boot/grub/grub.cfg
```

# Kernel modules

**Configuring the modules**

🗒️ **NOTE:**
Hardware modules are optional to be listed manually. **udev** will normally load all hardware modules that are detected to be connected in most cases. However, it is not harmful for automatically detected modules to be listed. Sometimes exotic hardware requires help to load their drivers.

List the modules that need to be loaded automatically in /etc/modules-load.d/*.conf files one module per line. Extra options for the modules, if necessary, should be set in /etc/modprobe.d/*.conf files.

To view all available modules, run the following **find** command. Don't forget to substitute "`<kernel version>`" with the version of the kernel just compiled:

```
(chroot) livecd /usr/src/linux # find /lib/modules/<kernel version>/ -type f -iname '*.o' -or -iname '*.ko' | less
```

For instance, to automatically load the 3c59x.ko module (which is the driver for a specific 3Com network card family), edit the /etc/modules-load.d/network.conf file and enter the module name in it. The actual file name is insignificant to the loader.

```
(chroot) livecd /usr/src/linux # mkdir -p /etc/modules-load.d
(chroot) livecd /usr/src/linux # vi /etc/modules-load.d/network.conf
```

```
FILE /etc/modules-load.d/network.confForce loading 3c59x module
3c59x
```

Continue the installation with [Configuring the system](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/System).

# Configuring the system

**Creating the fstab file**

The /etc/fstab file uses a table-like syntax. Every line consists of six fields, separated by whitespace (space(s), tabs or a mixture). Each field has its own meaning:

1.  The first field shows the block special device or remote filesystem to be mounted. Several kinds of device identifiers are available for block special device nodes, including paths to device files, filesystem labels and UUIDs, and partition labels and UUIDs.
2.  The second field shows the mount point at which the partition should be mounted.
3.  The third field shows the filesystem used by the partition.
4.  The fourth field shows the mount options used by mount when it wants to mount the partition. As every filesystem has its own **mount** options, users are encouraged to read the mount man page (**man mount**) for a full listing. Multiple mount options are comma-separated.
5.  The fifth field is used by dump to determine if the partition needs to be dumped or not. This can generally be left as 0 (zero).
6.  The sixth field is used by **fsck** to determine the order in which filesystems should be checked if the system wasn't shut down properly. The root filesystem should have 1 while the rest should have 2 (or 0 if a filesystem check isn't necessary).

**Important**
The default /etc/fstab file provided by Gentoo is not a valid fstab file but instead more of a template.

**Filesystem labels and UUIDs**
Both MBR (BIOS) and GPT include support for filesystem labels and filesystem UUIDs. These attributes can be defined in /etc/fstab as alternatives for the **mount** command to use when attempting to find and mount block devices. Filesystem labels and UUIDs are identified by the LABEL and UUID prefix and can be viewed with the **blkid** command:

Use the **blkid** command as reference to create the /etc/fstab file:

```
(chroot) livecd /usr/src/linux # cd
(chroot) livecd ~ # blkid > blkid.chroot-livecd.txt
```

🗒️ **NOTE:**
Because of uniqueness, readers that are using an MBR-style partition table are recommended to use UUIDs over labels to define mountable volumes in /etc/fstab.

In the remainder of the text, we use the default /dev/sd* block device files as partition.

Below is a more elaborate example of an /etc/fstab file:

```
/dev/sda1   /boot        ext2    defaults,noatime     0 2
/dev/sda2   none         swap    sw                   0 0
/dev/sda3   /            ext4    noatime              0 1
  
/dev/cdrom  /mnt/cdrom   auto    noauto,user          0 0
```

With the blkid output extracted previously, open it plus the /etc/fstab file, and create the fstab:

🗒️ **NOTE:**
With the following `Vi` usage, use `:n#X` to move from one file to another where the `X` is the order number of the file (1 or 2).

```
(chroot) livecd ~ # vi blkid.chroot-livecd.txt /etc/fstab
```

**Important**
Double-check the /etc/fstab file, save and quit to continue.

# systemd

[systemd](https://wiki.gentoo.org/wiki/Systemd) is a modern SysV-style init and [rc](https://wiki.gentoo.org/wiki/Rc) replacement for Linux systems. It is supported in Gentoo as an alternative init system.

**/etc/mtab**

Upstream only supports the /etc/mtab file being a symlink to /proc/self/mounts. Not creating this symlink will also cause problems with **mount** ([bug #434090](https://bugs.gentoo.org/show_bug.cgi?id=434090)) and **df** ([bug #477240](https://bugs.gentoo.org/show_bug.cgi?id=477240)). In the past some utilities wrote information (like mount options) into /etc/mtab and thus it was supposed to be a regular file. Nowadays all software is supposed to avoid this problem. Still, before switching the file to become a symbolic link, please check [bug #477498](https://bugs.gentoo.org/show_bug.cgi?id=477498) to be sure that the system is not affected by any reported regressions.

To create the symlink, run:

```
livecd ~ # ln -sf /proc/self/mounts /etc/mtab
livecd ~ # ls -l /etc/mtab 
lrwxrwxrwx 1 root root 17 May 22 10:28 /etc/mtab -> /proc/self/mounts
```

**Default: GRUB2**

By default, the majority of Gentoo systems now rely upon [GRUB](https://wiki.gentoo.org/wiki/GRUB) (found in the [sys-boot/grub](https://packages.gentoo.org/packages/sys-boot/grub) package), which is the direct successor to [GRUB Legacy](https://wiki.gentoo.org/wiki/GRUB_Legacy). With no additional configuration, GRUB2 gladly supports older BIOS ("pc") systems. With a small amount of configuration, necessary before build time, GRUB2 can support more than a half a dozen additional platforms. For more information, consult the [Prerequisites section](https://wiki.gentoo.org/wiki/GRUB2#Prerequisites) of the [GRUB2](https://wiki.gentoo.org/wiki/GRUB2) article.

**Emerge**

When using an older BIOS system supporting only MBR partition tables, no additional configuration is needed in order to emerge GRUB:

```
livecd ~ # emerge -av sys-boot/grub
```

A note for UEFI users: running the above command will output the enabled GRUB_PLATFORMS values before emerging. When using UEFI capable systems, users will need to ensure `GRUB_PLATFORMS="efi-64"` is enabled (as it is the case by default). If that is not the case for the setup, `GRUB_PLATFORMS="efi-64"` will need to be added to the /etc/portage/make.conf file before emerging GRUB2 so that the package will be built with EFI functionality:

```
livecd ~ # echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf
livecd ~ # emerge -a sys-boot/grub:2
```

If GRUB2 was somehow emerged without enabling `GRUB_PLATFORMS="efi-64"`, the line (as shown above) can be added to make.conf then and dependencies for the [world package set](https://wiki.gentoo.org/wiki/World_set_%28Portage%29 "https://wiki.gentoo.org/wiki/World_set_(Portage)") re-calculated by passing the `--update --newuse` options to **emerge**:

```
livecd ~ # emerge -auNv sys-boot/grub:2
```

The GRUB2 software has now been merged to the system, but not yet installed.

**Install**

Next, install the necessary GRUB2 files to the /boot/grub/ directory via the **grub-install** command. Presuming the first disk (the one where the system boots from) is /dev/sda, one of the following commands will do:

- When using BIOS:

```
livecd ~ # grub-install /dev/sda
Installing for i386-pc platform.
Installation finished. No error reported.
```

- When using UEFI:

**Important**
Make sure the EFI system partition has been mounted before running **grub-install**. It is possible for **grub-install** to install the GRUB EFI file (grubx64.efi) into the wrong directory **without** providing any indication the wrong directory was used.

```
livecd ~ # grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Gentoo
```

🗒️ **NOTE:**
Modify the `--efi-directory` option to the root of the EFI System Partition. This is necessary if the /boot partition was not formatted as a FAT variant.
Important
If grub_install returns an error like `Could not prepare Boot variable: Read-only file system`, it may be necessary to remount the efivars special mount as read-write in order to succeed:

```
livecd ~ # mount -o remount,rw /sys/firmware/efi/efivars
```

Some motherboard manufacturers seem to only support the /efi/boot/ directory location for the .EFI file in the EFI System Partition (ESP). The GRUB installer can perform this operation automatically with the `--removable` option. Verify the ESP is mounted before running the following commands. Presuming the ESP is mounted at /boot (as suggested earlier), execute:

```
root #grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Gentoo --removable
```

This creates the default directory defined by the UEFI specification, and then copies the grubx64.efi file to the 'default' EFI file location defined by the same specification.

**Configure**

Next, generate the GRUB2 configuration based on the user configuration specified in the /etc/default/grub file and /etc/grub.d scripts. In most cases, no configuration is needed by users as GRUB2 will automatically detect which kernel to boot (the highest one available in /boot/) and what the root file system is. It is also possible to append kernel parameters in /etc/default/grub using the GRUB\_CMDLINE\_LINUX variable.

Add the systemd support to GRUB2:

```
GRUB_CMDLINE_LINUX="init=/usr/lib/systemd/systemd"
```

To avoid the message below when generating the GRUB2:

```
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
```

Do the following:

```
livecd ~ # chmod 644 /etc/grub.d/30_os-prober
```

Then, copy these 2 lines in the /etc/default/grub:

```
# Disable os-prober
GRUB_DISABLE_OS_PROBER=true
```

🗒️ **NOTE:**
The output of the command must mention that at least one Linux image is found, as those are needed to boot the system. If an initramfs is used or **genkernel** was used to build the kernel, the correct initrd image should be detected as well. If this is not the case, go to /boot/ and check the contents using the **ls** command. If the files are indeed missing, go back to the kernel configuration and installation instructions.

To generate the final GRUB2 configuration, run the **grub-mkconfig** command:

```
livecd ~ # grub-mkconfig -o /boot/grub/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.12.5-gentoo
done
```

💡 **Tip:**
The os-prober utility can be used in conjunction with GRUB2 to detect other operating systems from attached drives. Windows 7, 8.1, 10, and other distributions of Linux are detectable. Those desiring dual boot systems should emerge the [sys-boot/os-prober](https://packages.gentoo.org/packages/sys-boot/os-prober) package then re-run the **grub-mkconfig** command (as seen above). If detection problems are encountered be sure to read the [GRUB2](https://wiki.gentoo.org/wiki/GRUB2) article in its entirety before asking the Gentoo community for support.

# Configuration

**Root password**

Set the root password using the **passwd** command.

```
livecd ~ # passwd 

You can now choose the new password or passphrase.

A valid password should be a mix of upper and lower case letters, digits, and
other characters.  You can use a password containing at least 7 characters
from all of these classes, or a password containing at least 8 characters
from just 3 of these 4 classes.
An upper case letter that begins the password and a digit that ends it do not
count towards the number of character classes used.

A passphrase should be of at least 3 words, 11 to 72 characters long, and
contain enough different characters.

Alternatively, if no one else can see your terminal now, you can pick this as
your password: "flow-length=toe".

Enter new password: 
Re-type new password: 
passwd: password updated successfully
```

The root Linux account is an all-powerful account, so pick a strong password. Later an additional regular user account will be created for daily operations.

**Network**

[iwd](https://wiki.gentoo.org/wiki/Iwd) (iNet Wireless Daemon) is an up-and-coming wireless daemon for Linux. It is written by [Intel](https://wiki.gentoo.org/wiki/Category:Intel) and aims to replace [wpa_supplicant](https://wiki.gentoo.org/wiki/Wpa_supplicant).

```
USE="... networkmanager iwd -wpa_supplicant"
```

After changing use flags run the following command to update the system so the changes take effect:

```
(chroot) livecd /usr/src/linux # emerge -avuDN @world
```

Then, install [NetworkManager](https://wiki.gentoo.org/wiki/NetworkManager) and [iwd](https://wiki.gentoo.org/wiki/Iwd):

```
(chroot) livecd /usr/src/linux # emerge -a net-misc/networkmanager net-wireless/iwd
```

Before the fist rebooting, enable `sshd`, `NetworkManager` and `iwd` for remote support:

```
(chroot) livecd /usr/src/linux # systemctl enable sshd
(chroot) livecd /usr/src/linux # systemctl enable NetworkManager
(chroot) livecd /usr/src/linux # systemctl enable NetworkManager-wait-online.service
(chroot) livecd /usr/src/linux # systemctl enable iwd
```

To set the hostname, create/edit /etc/hostname and simply provide the desired hostname.

```
(chroot) livecd /usr/src/linux # echo "gentoo-linux" > /etc/hostname
```

**Rebooting the system**

Exit the chrooted environment and unmount all mounted partitions. Then type in that one magical command that initiates the final, true test: **reboot**.

```
(chroot) livecd /usr/src/linux # exit
livecd ~ # cd
livecd ~ # umount -l /mnt/gentoo/dev{/shm,/pts,}
livecd ~ # umount -R /mnt/gentoo
livecd ~ # reboot
```

Do not forget to remove the bootable CD, otherwise the CD might be booted again instead of the new Gentoo system.

Once rebooted in the freshly installed Gentoo environment, finish up with [Finalizing the Gentoo installation](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Finalizing).

**Booting with systemd**

[systemd](https://wiki.gentoo.org/wiki/Systemd) supports a few system configuration files to set the most basic system details.

🗒️ **NOTE:**
While some system configuration parameters can be updated by modifying the appropriate configuration files, most settings are managed using utilities that require systemd to be running. In this case, it is safe to reboot the computer with systemd and use the **hostnamectl**, **localectl**, and **timedatectl** utilities as required.

**Connecting to internet with iwctl**
**iwctl** is iwd's tool to control iwd. It supports both a command line interface and an interactive mode. A complete command line would be iwctl station list to see what adapters you might be able to use:

```
gentoo-linux ~ # iwctl station list
                            Devices in Station Mode                            
--------------------------------------------------------------------------------
  Name                State          Scanning
--------------------------------------------------------------------------------
  wlan0               disconnected           
```

Use the following command to connect the WiFi network:

```
gentoo-linux ~ # iwctl station wlan0 connect "network_name"
Type the network passphrase for network_name psk.
Passphrase: ****
```

**Machine ID**

Create a machine ID for journaling to work. This can be done through the following command:

```
gentoo-linux ~ # systemd-machine-id-setup
```

🗒️ **NOTE:**
The command `systemd-machine-id-setup` also has an impact on the `systemd-networkd` service. If you don't run this command, strange behavior like network interfaces not coming UP or network addresses not being applied will occur.

**Hostname**

**When booted using systemd**, a tool called **hostnamectl** exists for editing /etc/hostname and /etc/machine-info. To change the hostname if needed, run:

```
gentoo-linux ~ # hostnamectl set-hostname <HOSTNAME>
```

Refer to man hostnamectl for more options.

**Locale**

Usually, locales will be properly migrated from OpenRC when installing systemd. When required, the locale can be set in /etc/locale.conf as per the Gentoo handbook instructions:

```
LANG="en_US.utf8"
```

Once booted with systemd, the tool localectl is used to set locale and console or X11 keymaps. To change the system locale, run the following command:

```
gentoo-linux ~ # localectl set-locale LANG=en_US.utf8
```

To change the virtual console keymap:

```
gentoo-linux ~ # localectl set-keymap us
```

And finally, to set the X11 layout:

```
gentoo-linux ~ # localectl set-x11-keymap us
```

If needed the model, variant and options can be specified as well:

```
gentoo-linux ~ # localectl set-x11-keymap <LAYOUT> <MODEL> <VARIANT> <OPTIONS>
```

Check the **localectl status**:

```
gentoo-linux ~ # localectl status
   System Locale: LANG=en_US.UTF-8
                  LC_COLLATE=C.UTF-8
       VC Keymap: us
      X11 Layout: us
```

After doing any of the above, update the environment so the changes will take effect:

```
gentoo-linux ~ # env-update && source /etc/profile
>>> Regenerating /etc/ld.so.cache...
```

**Time and date**

Time, date, and timezone can be set using the **timedatectl** utility. That will also allow users to set up synchronization without needing to rely on [net-misc/ntp](https://packages.gentoo.org/packages/net-misc/ntp) or other providers than systemd's own implementation.

To learn how to use **timedatectl** simply run:

```
gentoo-linux ~ # timedatectl --help
```

**Automatic module loading**

Automatic module loading is configured in a different file, or rather directory of files. The configuration files are stored in /etc/modules-load.d. On boot every file with a list of modules will be loaded. The file format is a list of modules separated by newlines and can have any name as long as it ends with .conf. The module loading can be separated by program, service or whatever way that fits personal preference. An example virtualbox.conf is listed below:

```
FILE /etc/modules-load.d/virtualbox.confExample file for the virtualbox modules
vboxdrv
vboxnetflt
vboxnetadp
vboxpci
```

**NetworkManager**

Often NetworkManager is used to configure network settings. For that purpose, simply run the following command when using a graphical desktop:

```
gentoo-linux ~ # nm-connection-editor
```

If that is not the case and the network needs to be configured from console, give [nmcli](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Networking_Guide/sec-Using_the_NetworkManager_Command_Line_Tool_nmcli.html) a try, or follow a guided configuration process through **nmtui**:

```
gentoo-linux ~ # nmtui
```

[nmtui](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Networking_Guide/sec-Networking_Config_Using_nmtui.html) is a curses frontend that will guide the user in the process while running in console mode.

**The hosts file**

Next inform Linux about the network environment. This is defined in /etc/hosts and helps in resolving host names to IP addresses for hosts that aren't resolved by the nameserver.

```
gentoo-linux ~ # vi /etc/hosts
```

```
# This defines the current system and must be set
127.0.0.1     gentoo-linux.localdomain gentoo-linux localhost
  
# Optional definition of extra systems on the network
192.168.1.100   server_name
```

Save and exit the editor to continue.

**Adding a user for daily use**

Working as root on a Unix/Linux system is dangerous and should be avoided as much as possible. Therefore it is strongly recommended to add a user for day-to-day use.

The groups the user is member of define what activities the user can perform. The following table lists a number of important groups:

| **Group** | **Description** |
| --- | --- |
| audio | Be able to access the audio devices. |
| cdrom | Be able to directly access optical devices. |
| floppy | Be able to directly access floppy devices. |
| games | Be able to play games. |
| portage | Be able to access portage restricted resources. |
| usb | Be able to access USB devices. |
| video | Be able to access video capturing hardware and doing hardware acceleration. |
| wheel | Be able to use su. |

Let's create an admin user:

```
gentoo-linux ~ # useradd -m -g users -G wheel,audio,video,portage -s /bin/bash -m -c "Nelson Fonseca" nelson && passwd nelson
```

Install app-admin/sudo to be able to run admin commands with this user:

```
gentoo-linux ~ # emerge -a app-admin/sudo
```

Use **visudo** command to modify the wheel group permissions if needed.

**Removing tarballs**

With the Gentoo installation finished and the system rebooted, if everything has gone well, we can now remove the downloaded stage3 tarball from the hard disk. Remember that they were downloaded to the / directory.

```
gentoo-linux ~ # rm -v /stage3-*.tar.*
```

# Welcome to Portage

Portage is one of Gentoo's most notable innovations in software management. With its high flexibility and enormous amount of features it is frequently seen as the best software management tool available for Linux.

Portage is completely written in [Python](https://www.python.org/) and [Bash](https://www.gnu.org/software/bash) and therefore fully visible to the users as both are scripting languages.

Most users will work with Portage through the emerge tool. This chapter is not meant to duplicate the information available from the emerge man page. For a complete rundown of emerge's options, please consult the man page:

```
gentoo-linux ~ # man emerge
```

**Updating the Gentoo repository**

The Gentoo repository is usually updated with rsync, a fast incremental file transfer utility. Updating is fairly simple as the emerge command provides a front-end for rsync:

```
gentoo-linux ~ # emerge --sync
```

Sometimes firewall restrictions apply that prevent rsync from contacting the mirrors. In this case, update the Gentoo repository through Gentoo's daily generated snapshots. The **emerge-webrsync** tool automatically fetches and installs the latest snapshot on the system:

```
gentoo-linux ~ # emerge-webrsync
```

An additional advantage of using emerge-webrsync is that it allows the administrator to only pull in Gentoo repository snapshots that are signed by the Gentoo release engineering GPG key. More information on this can be found in the Portage features section on [fetching validated Gentoo repository snapshots](https://wiki.gentoo.org/wiki/Handbook:Parts/Working/Features#Validated_Gentoo_repository_snapshots).

**Updating the system**

To keep the system in perfect shape (and not to mention install the latest security updates) it is necessary to update the system regularly. Since Portage only checks the ebuilds in the Gentoo repository, the first thing to do is to update this repository. When the Gentoo repository is updated, the system can be updated using **emerge --update @world**. In the next example, the `--ask` option is also used which will tell Portage to display the list of packages it wants to upgrade and ask for confirmation:

```
gentoo-linux ~ # emerge -ua @world
```

Portage will then search for newer version of the applications that are installed. However, it will only verify the versions for the applications that are explicitly installed (the applications listed in /var/lib/portage/world) - it does not thoroughly check their dependencies. To update the dependencies of those packages as well, add the `--deep` option:

```
gentoo-linux ~ # emerge -uD @world
```

Still, this does not mean all packages: some packages on the system are needed during the compile and build process of packages, but once that package is installed, these dependencies are no longer required. Portage calls those build dependencies. To include those in an update cycle, add `--with-bdeps=y`:

```
gentoo-linux ~ # emerge -uD --with-bdeps=y @world
```

Since security updates also happen in packages that are not explicitly installed on the system (but that are pulled in as dependencies of other programs), it is recommended to run this command once in a while.

If the USE settings of the system have been altered, it is recommended to add `--newuse` as well. Portage will then verify if the change requires the installation of new packages or recompilation of existing ones:

```
gentoo-linux ~ # emerge -uDN --with-bdeps=y @world
```

**Metapackages**

Some packages in the Gentoo repository don't have any real content but are used to install a collection of packages. For instance, the [kde-plasma/plasma-meta](https://packages.gentoo.org/packages/kde-plasma/plasma-meta) package will install the KDE Plasma desktop on the system by pulling in various Plasma-related packages as dependencies.

To remove such a package from your system, running **emerge --deselect** on the package won't have much effect as the dependencies remain on the system.

Portage has the functionality to remove orphaned dependencies as well, but since the availability of software is dynamically dependent it is important to first update the entire system fully, including the new changes applied when changing USE flags. After this one can run **emerge --depclean** to remove the orphaned dependencies. When this is done, it might be necessary to rebuild the applications that were dynamically linked to the now-removed software titles but don't require them anymore, although recently support for this has been added to Portage.

All this is handled with the following two commands:

```
gentoo-linux ~ # emerge -uDN @world
gentoo-linux ~ # emerge -a --depclean
```

**Searching for software**

There are multiple ways to search through the Gentoo repository for software. One way is through emerge itself. By default, **emerge --search** returns the names of packages whose title matches (either fully or partially) the given search term.

For instance, to search for all packages who have "pdf" in their name:

```
gentoo-linux ~ # emerge --search pdf
```

To search through the descriptions as well, use the --searchdesc (or -S) option:

```
gentoo-linux ~ # emerge --searchdesc pdf
```

Or simply run the `emerge -s pdf` command.

**Installing software**

When a software title has been found, then the installation is just one **emerge** command away. For instance, to install gnumeric:

```
gentoo-linux ~ # emerge --ask app-office/gnumeric
```

Since many applications depend on each other, any attempt to install a certain software package might result in the installation of several dependencies as well. Don't worry, Portage handles dependencies well. To find out what Portage would install, add the `--pretend` option. For instance:

```
gentoo-linux ~ # emerge --pretend gnumeric
```

During the installation of a package, Portage will download the necessary source code from the Internet (if necessary) and store it by default in /var/cache/distfiles/. After this it will unpack, compile and install the package. To tell Portage to only download the sources without installing them, add the `--fetchonly` option to the emerge command:

```
gentoo-linux ~ # emerge --fetchonly gnumeric
```

**Finding installed package documentation**

Many packages come with their own documentation. Sometimes, the doc USE flag determines whether the package documentation should be installed or not. To see if the doc USE flag is used by a package, use **emerge -vp category/package**:

```
gentoo-linux ~ # emerge -vp media-libs/alsa-lib
These are the packages that would be merged, in order:
 
Calculating dependencies... done!
[ebuild   R    ] media-libs/alsa-lib-1.1.3::gentoo  USE="python -alisp -debug -doc" ABI_X86="(64) -32 (-x32)" PYTHON_TARGETS="python2_7" 0 KiB
```

The best way of enabling the doc USE flag is doing it on a per-package basis via /etc/portage/package.use, so that only the documentation for the wanted packages is installed. For more information read the [USE flags](https://wiki.gentoo.org/wiki/Handbook:AMD64/Working/USE) section.

Once the package installed, its documentation is generally found in a subdirectory named after the package in the /usr/share/doc/ directory:

```
gentoo-linux ~ # ls -l /usr/share/doc/alsa-lib-1.1.3
total 16
-rw-r--r-- 1 root root 3098 Mar  9 15:36 asoundrc.txt.bz2
-rw-r--r-- 1 root root  672 Mar  9 15:36 ChangeLog.bz2
-rw-r--r-- 1 root root 1083 Mar  9 15:36 NOTES.bz2
-rw-r--r-- 1 root root  220 Mar  9 15:36 TODO.bz2
```

A more sure way to list installed documentation files is to use **equery**'s `--filter` option. equery is used to query Portage's database and comes as part of the [app-portage/gentoolkit](https://packages.gentoo.org/packages/app-portage/gentoolkit) package:

```
gentoo-linux ~ # equery files --filter=doc alsa-lib
 * Searching for alsa-lib in media-libs ...
 * Contents of media-libs/alsa-lib-1.1.3:
/usr/share/doc/alsa-lib-1.1.3/ChangeLog.bz2
/usr/share/doc/alsa-lib-1.1.3/NOTES.bz2
/usr/share/doc/alsa-lib-1.1.3/TODO.bz2
/usr/share/doc/alsa-lib-1.1.3/asoundrc.txt.bz2
```

The `--filter` option can be used with other rules to view the install locations for many other types of files. Additional functionality can be reviewed in **equery**'s man page: **man 1 equery**.

These 2 tools below are very useful to consult package details:

```
gentoo-linux ~ # emerge -a app-portage/genlop app-portage/gentoolkit
```

🗒️ **NOTE:**
With **equery** command, we can check details such as the dependencies from a package, and with the **genlop** command, we can find what was the last time duration of a package's compilation.

**Examples:**

```
gentoo-linux ~ # equery depends app-editors/vim
 * These packages depend on app-editors/vim:
app-vim/gentoo-syntax-20210428 (>=app-editors/vim-7.3)
dev-util/ninja-1.10.2-r1 (vim-syntax ? app-editors/vim)
virtual/editor-0-r3 (app-editors/vim)
virtual/pager-0 (app-editors/vim[vim-pager])
```

```
gentoo-linux ~ # genlop -t app-editors/vim | tail -n 3
     Wed May 26 10:22:24 2021 >>> app-editors/vim-8.2.0814-r100
       merge time: 2 minutes and 26 seconds.
```

# Xorg

## The X.org project

The [X.org](http://www.x.org/) project created and maintains a freely redistributable, open-source implementation of the X11 system. It is an open source X11-based desktop infrastructure.

Xorg provides an interface between your hardware and the graphical software you want to run. Besides that, Xorg is also fully network-aware, meaning you are able to run an application on one system while viewing it on a different one.

## Installation

**USE flags**
Portage knows the X USE flag for enabling support for X in other packages (default in all desktop [profiles](https://wiki.gentoo.org/wiki/Profile_%28Portage%29 "https://wiki.gentoo.org/wiki/Profile_(Portage)")). Make sure this USE flag is added to the USE flag list to ensure X compatibility system wide:

```
FILE /etc/portage/make.conf
USE="... X ..."
```

Check all the options available and choose those which apply to the system. This example is for a system with a keyboard, mouse, Synaptics touchpad, and a Radeon video card.

```
gentoo-linux ~ # emerge -pv x11-base/xorg-drivers

These are the packages that would be merged, in order:

Calculating dependencies... done!
[ebuild  N     ] x11-misc/util-macros-1.19.3::gentoo  83 KiB
[ebuild  N     ] x11-misc/xbitmaps-1.1.2-r1::gentoo  127 KiB
[ebuild  N     ] x11-misc/xkeyboard-config-2.32::gentoo  1,702 KiB
[ebuild  N     ] x11-libs/libpciaccess-0.16::gentoo  USE="zlib" ABI_X86="(64) -32 (-x32)" 359 KiB
[ebuild  N     ] x11-libs/libxkbfile-1.1.0::gentoo  ABI_X86="(64) -32 (-x32)" 357 KiB
[ebuild  N     ] x11-libs/libXtst-1.2.3-r2::gentoo  USE="-doc" ABI_X86="(64) -32 (-x32)" 315 KiB
[ebuild  N     ] x11-apps/iceauth-1.0.8-r1::gentoo  135 KiB
[ebuild  N     ] x11-apps/rgb-1.0.6-r1::gentoo  136 KiB
[ebuild  N     ] x11-libs/libXfont2-2.0.4::gentoo  USE="bzip2 ipv6 truetype -doc" 502 KiB
[ebuild  N     ] media-libs/libepoxy-1.5.8::gentoo  USE="X egl -test" ABI_X86="(64) -32 (-x32)" 325 KiB
[ebuild  N     ] x11-apps/xrdb-1.2.0::gentoo  139 KiB
[ebuild  N     ] x11-apps/xkbcomp-1.4.5::gentoo  246 KiB
[ebuild  N     ] x11-apps/xinit-1.4.1-r1::gentoo  USE="-twm" 173 KiB
[ebuild  N     ] x11-base/xorg-server-1.20.11:0/1.20.11::gentoo  USE="ipv6 systemd udev xorg -debug -dmx -doc (-elogind) -kdrive -minimal (-selinux) -suid -test -unwind -wayland -xcsecurity -xephyr -xnest -xvfb" 6,325 KiB
[ebuild  N     ] x11-base/xorg-drivers-1.20-r2::gentoo  INPUT_DEVICES="libinput synaptics -elographics -evdev -joystick -vmmouse -void -wacom" VIDEO_CARDS="nouveau -amdgpu -ast -dummy -fbdev (-freedreno) (-geode) -glint -i915 -i965 -intel -mga -nv -nvidia (-omap) -qxl -r128 -radeon -radeonsi -siliconmotion (-tegra) (-vc4) -vesa -via -virtualbox -vmware" 0 KiB
[ebuild  N     ] x11-drivers/xf86-input-libinput-1.0.1::gentoo  371 KiB
[ebuild  N     ] x11-drivers/xf86-video-nouveau-1.0.17::gentoo  618 KiB
[ebuild  N     ] x11-drivers/xf86-input-synaptics-1.9.1::gentoo  492 KiB

Total: 18 packages (18 new), Size of downloads: 12,396 KiB
```

After setting all the necessary variables Xorg can be installed:

```
gentoo-linux ~ # emerge -a x11-base/xorg-server
```

🗒️ **NOTE:**
The [x11-base/xorg-x11](https://packages.gentoo.org/packages/x11-base/xorg-x11) meta-package could be installed instead of the more lightweight [x11-base/xorg-server](https://packages.gentoo.org/packages/x11-base/xorg-server). Functionally [x11-base/xorg-x11](https://packages.gentoo.org/packages/x11-base/xorg-x11) and [x11-base/xorg-server](https://packages.gentoo.org/packages/x11-base/xorg-server) are the same, however [x11-base/xorg-x11](https://packages.gentoo.org/packages/x11-base/xorg-x11) brings in many more packages that most systems will probably not require. Additional packages include a large assortment of fonts in many languages. They are not necessary for a working X11 framework.

When the installation is finished, some environment variables will need to re-initialized before continuing. Source the profile with this command:

```
gentoo-linux ~ # env-update
gentoo-linux ~ # source /etc/profile
```

# KDE-Plasma Minimal

Install the following packages for KDE Plasma only:

```
emerge -a kde-plasma/plasma-desktop kde-plasma/bluedevil kde-plasma/breeze kde-plasma/breeze-gtk kde-plasma/drkonqi kde-plasma/kactivitymanagerd kde-plasma/kde-gtk-config kde-plasma/kdecoration kde-plasma/kdeplasma-addons kde-plasma/kgamma kde-plasma/khotkeys kde-plasma/kscreen kde-plasma/kscreenlocker kde-plasma/ksshaskpass kde-plasma/ksysguard kde-plasma/kwallet-pam kde-plasma/kwin kde-plasma/libkscreen kde-plasma/libksysguard kde-plasma/libkworkspace kde-plasma/plasma-disks kde-plasma/plasma-firewall kde-plasma/plasma-integration kde-plasma/plasma-nm kde-plasma/plasma-pa kde-plasma/plasma-systemmonitor kde-plasma/plasma-vault kde-plasma/plasma-workspace kde-plasma/powerdevil kde-plasma/sddm-kcm kde-plasma/systemsettings kde-plasma/xdg-desktop-portal-kde kde-plasma/xembed-sni-proxy kde-apps/ark kde-apps/audiocd-kio kde-apps/dolphin kde-apps/dragon kde-apps/ffmpegthumbs kde-apps/filelight kde-apps/gwenview kde-apps/k3b kde-apps/kamera kde-apps/kate kde-apps/kbackup kde-apps/kcalc kde-apps/kcron kde-apps/kdenetwork-filesharing kde-apps/kdenlive kde-apps/kdf kde-apps/kio-extras kde-apps/kolourpaint kde-apps/kompare kde-apps/konsole kde-apps/krdc kde-apps/krfb kde-apps/kwalletmanager kde-apps/okular kde-apps/print-manager kde-apps/spectacle kde-apps/ksystemlog kde-apps/thumbnailers kde-apps/yakuake kde-apps/zeroconf-ioslave kde-misc/kdeconnect
```

Install ohter packages:

```
emerge -a app-office/libreoffice www-client/firefox-bin mail-client/thunderbird net-im/telegram-desktop www-client/opera app-arch/unrar app-arch/unzip app-arch/p7zip media-gfx/gimp net-misc/anydesk sys-block/partitionmanager sys-fs/ntfs3g sys-fs/hfsutils app-emulation/qemu app-emulation/libvirt app-emulation/virt-manager media-sound/elisa media-sound/pavucontrol x11-misc/xdg-user-dirs sys-apps/{usbutils,hwinfo} x11-apps/xrandr sys-apps/udevil
```