pwd
whoami
ip addr show
free -m 
df -h  
cat /etc/host
findmnt

## bash shell
 `<` stdin   `>` stdout


1. `>` 
2. `>>` 
3. `2>/dev/null`
4. `<`

## FHS

## vim
- `Esc`
- ` i,a`
- `o`
- `:wq`
- `:q` 
- `dd`
- `yy`
- `p`
- `v`
- `u`
- `gg`
- `G`
- `/txt`
- `?txt`
- `^`
- `$`
- `:%s/old/new/g`

## Globbing

- `ls host*`
- `ls ?ost`
- `ls [hm]ost`
- `ls [!hm]ost`
- `ls script[0-9][0-9]` 
- `touch script{0..9}`

## cockpit.socket
CentOS 监控工具

## File Management Tools

- `ls`
- `mkdir`
- `cp`
- `mv`
- `rmdir`
- `rm `
- `ls`
- `which`
- `locate / updatedb`
- `find`
- `mnt`
- `link`
- `tar cvf|tvf|xvf`

## Text Tools

- `more`
- `less`
- `tail|head -n nn`
- `tac`
- `cut -f 1 -d : /etc/passwd | sort  -n | tr [a-z] [A-Z]`
- `sort`
- `tr`
- `grep -lR root /etc/ 2>/dev/null`
- `\<` beginning of word and `\>` end of word

## Connect to a Server

- `chvt` change virtual terminal

## Users

- `useradd -D`
- `usermod`
- `userdel`
- `passwd`
- `cat /etc/default/useradd`
- `vim /etc/login.defs`
- `cd  /etc/skel`
- `/etc/passwd`
- `/etc/shadow`
- `/etc/group`
- `groupadd sales`
- `groupmod sales`
- `groupdel sales`
- `lid -g groupname`
- `chage|passwd`


## File ownership 
- `chown user[:group] file`  `chown linda:student newfile`
- `chgrp group file`
- `chmod`
- `umask` modify default dir(777) or file(666) permision
- SUID| SGID | Stricky Bit


- SUID
    - chmod 4770 myfile
    - chmod u+s myfile
- GID
    - chmod 2770 mydir
    - chmod g+s mydir 
- Sticky bit
    - chmod 1770 mydir
    - chomd +t  


## Networking

- `ip link show`
- `ip addr show`
- `lo` 

### lo config

- `ip addr add dev ens33 ip`
- `ip route show`
- `nmcli con|`
- `rpm -qa | grep bash-completion` 
- `nmcli connection add ifname enp0s3333 con-name enp0s3333 ipv4.addresses 10.0.2.16/24 ipv4.dns 8.8.8.8 ipv4.gateway 10.0.2.1 type ethernet`
- `nmtui`

`/etc/sysconfig/network-scripts/ifcfg-enp0s8` network card config   

### Network Testing Tools

- `ping`
- `dig` for dns

## Operation Running System

### Managing Processes

- `command &` start a job in the background
- To move a job to the background  
    - First, stop it using `Ctrl-Z`
    - Next, type `bg` to move it to the background 
- Use `jobs` for a complete overview of running jobs
- Use `fg [n]` to move the last job back to the foreground


#### ps 

- `ps -L` and `ps L` are diffrent
- `ps aux` 
- `ps -fax` 
- `ps -fU linda`
- `ps -f --forest -C sshd` shows a process tree for a specific process
- `ps L` shows format specifiers
- `ps -eo pid,ppid,user,cmd`

#### Memery Usage


#### CPU load

- `lscpu `
- `top`
- `nice | renice`


#### Tuned

## Mangering Software

### `rpm` package

#### set up local repository

install package form the RHEL 8 instaallation disk ISO image

- create an iso image : `dd if=/dev/sr0 of=/rhel8.iso bs=1M`
- create a dir /repo : `mkdir /repo`
- edit /etc/fstab and add the following line to the end
    - `/rhel8.iso /repo iso9660 defaults 0 0`
- use `mount -a ` to mount the ISO
- create the file /etc/yum.repos.d/appstream.repo


###### yum

- `yum search`
- `yum install`
- `yum remove`
- `yum update`
- `yum provides`
- `yum list`


###### yum module

- `yum module list`
- `yum module provides https`
- `yum module info php`
- `yum module info --profile php`
- `yum module list php`
- `yum module install php:7.1 | yum insall @php:7.1`
- `yum module install php:7.1/devel`
- `yum module distro-sync`
- `yum groups list`
- `yum groups list hidden`
- `yum groups info <groupname>`
- `yum groups install <groupname>`
- `yum history`
- `yum history undo `
- `yum update <pacakgename>`

##### rpm

- `rpm` is useful though to perform package queries
- `rmp -qf /any/file`
- `rpm -ql mypackage`
- `rmp -qc mypacakge`
- `rpm -qp --scripts mypackage-file.rmp ` 

### systemd

- managed items are called units
- different unit types are available
    - services
    - mounts
    - times
    - and many more
- `systemctl`
- `systemctl enabel unit` auto start unit
- config file in `/usr/lib/systemd/system` 
- custom unit files are in `/etc/systemd/system`
- run-time automatically generated unit files are in `/run/systemd`
- don't modify a unit file in `/usr/lib/systemd/system`, but create a custom file in `/etc/systemd/system` that is used as an overlay
- better : use `systemctl edit unit.service` to edit unit files
- use `systemctl show` to show available parameters
- using `systemctl-reload` may be required
 


``` 
systemctl cat vsftpd.service

systemctl  show vsftpd

systemctl edit vsftpd

[Service]
param=value

systemctl daemon-reload 

systemctl enable vsftpd
``` 


### Scheduling Tasks

- cron is a daemon that triggers jobs on a regular basis
- it works with diffrent configuration files that specify when a job should be started
- use it for regular re-occurring jobs, like backup jobs
- `at` is used for tasks that need to be started once
- systemd Timers provide a new alternative to Cron

#### Cron

- user-specific cron jobs ,created using `crontab -e`
- generic time-specific cron jobs in `/etc/cron.d`
- scripts, executed on an hourly, daily, weekly, monthly basis
- generic time-specific cron jobs in `/etc/crontab`(deprecated)

##### Anacron

- anacron is a service behind cron that takes care jobs are executed on a regular basis, but not at a specific time
- it takes care of the jobs in `/etc/cron.{hourly,daily,weekly,monthly}`
- configuration is in `/etc/anacrontab`

###### systemd timers

- systemd timers also allow for scheduling jobs at a regular basis, Cron however is still the standard
- read `man 7 systemd-timer ` for more information about systemd timers
- read `man 7 systemd-time` for specification of the time format to be used

###### `at`

- the `atd` service must be running to run once-only jobs using `at`
- use `at <time>` to schedule a job
    - type one or more job specifications in the at interactive shell
    - user ctrl-D to close this shell 
- use `atp` for a list of jobs currently scheduled
- use `atrm` to remove jobs from the list

###### `systemd-tmpfiles`

- the `/usr/lib/tmpfiles.d` directory manages settings for creating, deleting and cleaning up of temporary files
- the `systemd-tmpfiles-clean.timer` unit can be configured to automatically clean up temporary files.
    - it triggers the `systemd-tmpfiles-clean.timer.service`
    - this service runs `systemd-tmpfiles --clean`
- the `/usr/lib/tmpfiles.d/tmp.conf` file contains settings for the automatic tmp file cleanup
- when making modifications, copy the file to `/etc/tmpfils.d`
- after making modifications to this file, use `systemd-tmpfiles --clean /etc/tmpfiles.d/tmp.conf` to ensure the file does not contain any errors

## Configuring logging

`rsyslogd` -> `/var/log` work with `systemd` or `systemd-journalctl`(`journalctl`)

- `rsynlogd` service running 
- the main configuration file is `/etc/rsyslog.conf`
- snap-in files can be placed in `/etc/rsyslog.d`
- each logger line containe three items
    - facility: the specific facility that the log is created for
    - severity: the severity from which should be logged
    - destination: the file or other destination the log should be written to 
- log files normally are in `/vat/log`
- use the `logger` command to write messages to rsyslog manually

### facility

- `rsyslogd` is and must be backwards compatible with the archaic syslog service
- in syslog, a fixed number of facilities was defined, like kern, authprive, cron and more
- to work with services that don't have their own facility local{0..7} can be used
- because of the lack of facilities, some services take care of their own logging and don't use rsyslog

### `systemd-journald`

- `systemd-journald` is the log service that is a part of systemd
- it integrates well with `systemctl status <unit>` output
- alternatively ,the `journald` command can be used to read log entries in the journal 
- messages are logged also to rsyslogd , using the rsyslogd imjournal module
- to make the journal persistent ,use `mkdir /var/log/journal`

### keeping the system journal

- the journal is written to `/run/log/journal`, which is automatically cleared on system reboot
- eidt `/etc/systemd/journald.conf` to make the journal persistent across reboots
- set the `Storage` parameter in this file to the appropriate value
    - `persistent` will store the journal in the `/var/log/journal` directory. this directory will be created if it doesn't exist
    - `volatile` stores the journal only in `/run/log/journal`
    - `auto` will stores the journal only in `/var/log/journal` if that directory exists, and in `/run/log/journal` if no `/var/log/journal` exists

### systemd jouranl log rotation

- build-in log rotation for the journal runs monthly
- the journal cannot grow beyond 10% of the size of the file system it is on
- the journal will also make sure at least 15% of its file system will remain available as free space
- these settings can be changed through `/etc/systemd/journal.conf`

#### logrotate

- logrotate is started through cron.daily to ensure that log files don't grow too big
- main configuration is in `/etc/logrotate.conf` , snap-in files can be provided through `/etc/logrotate.d/`

## manage storage

BIOS MBR 4 partition
UEFI GRT 128 partition

#### storage options

- Partitions: the classical solution, use in all cases
    - use to allocate dedicated storage to specific types of data
- LVM logical Volumes 
    - userd at default installation of RHL
    - adds flexibility to storage (resize, snapshots and more)
- Stratis
    - next generation Volume Managing Filesystem that uses thin provisioning by default
    - Implemented in user space , which makes API access possible
- Virtual Data Optimzier
    - Focused on storing files in the most efficient way
    - Manages deduplicated and compressed storage pools

#### GPT and MBR Partitions

- Master Boot Record (MBR) is part of the 1981 PC specification
    - 512 bytes to store boot information
    - 64 byte to store partitions
    - place for 4 partitons only with a max, size of 2 TiB
    - to use more partitions , extended and logical partitions must be used
- GUID Partitions Table is a newer partition table (2010)
    - More space to store partitons
    - Used to overcome MBR limitations
    - 128 partitions max


#### create partitions with `parted`

- while creating a partion, you do not automatically create a file system
- the `parted` file system attribute only writes some unimportant file system metadata
- in RHEL, `parted` is the default utility
- Alternatively, use `fdisk` to work with MBR and `gdisk` to use GUID partitons

##### Procedure 

1. `parted /dev/sdb`
2. `print` will show if there is a current partition table
3. `mklabel msdos|gpt`
4. `mkpart part-type name fs-type start end`
    - `part-type`: applies to MBR only and sets primary, logical, or extended partition
    - `name`: arbitrary name, required for GPT
    - `fs-type`: does not modify the filesystem, but sets some irrelevant file system dependent metadata
    - `start end`: specify start and end, counting from the beginning of the disk
    - for instance: `mkpart primary 1024MiB 2048MiB`
5. alternatively, use `mkpart` in interactive mode
6. `print` to verify creation of the new partiton
7. `quit` to exit the parted shell
8. `udevadm settle` to ensure that the new partition device is created
9. `cat /proc/partitions` to verify the creation of the partiton


#### File system difference

- XFS is the default file system
    - fast and scalable
    - uses CoW to guarantee data integrity
    - size can be increased, not decreased
- Ext4 was default in RHEL 6 and is still used
    - backward compatible to Ext2
    - users journal to guarantee data integrity
    - size can be increased and decreased
- Other file systems are available but less common

#### Making and Mounting File System

- `mkfs.xfs` creates an XFS file system
- `mkfs.ext4`
- `mkfs.[Tab][Tab]` to show a list of available file systems
- Don't use `mkfs` as it will create an Ext2 file system!
- After making the file system, you can mount it in runtime using the `mount` command
- use `umount` to disconnect a device

#### use `/etc/fstab` to mount

- `/etc/fstab` is the main configuration file to persistently mount partitions
- `/etc/fstab` content is used to generate systemd mounts by the `systemd-fstab-generator` utility
- to update systemd, make sure to use `systemctl daemon-reload` after editing `/etc/fstab`

#### persistent device naming

in datacenter environments, block device names may change. different solutions exist for persistent namning

- UUID : a UUID is automatically generated for each device that device that contains a file system or anything similiar
- Label : wile creating the file system, the options -L can be used to set an arbitrary name that can be userd for mounting the fyle system
- Unique device names are created in `/dev/disk`

`tune2fs` tool to modify file system label 

#### manage systemd mount

- `/etc/fstab` mounts already are systemd mounts
- mounts can be created using systemd `.mount` files
- using `.mount` files allows you to be more spcific in defining dependencies
- use `systemctl cat tmp.mount` for an example

#### manage `xfs` file system

- the `xfsdump` utility can be used for creating backup of XFS fromatted devices and considers specific XFS attributes
    - `xfsdump` only works on a complete XFS device
    - `xfsdump` can make full backup (-l 0) or different levels of incremental backups
    - `xfsdump -l 0 -f /backupfiles/data.xfsdump /data` creates a full backup of the contents of the `/data` directory
- the `xfsrestore` command is used to restore a backup that was made with `xfsdump`
    - `xfsrestore -f /backupfiles/data.xfsdump /data`
- the `xfsrepair` command can be manually started to repair broken XFS file systems

#### Swap partition

- Swap is RAM that is emulated on disk
- All linux systems should have at least some swap
    - the amount of swap depends on the use of the server
- Swap can be created on ayn block device, includeing swap files
- while creating swap with `parted`, set file system to linux-swap
- after creating the swap partition, use `mkswap` to create the swap FS
- activate using `swapon`

### LVM Staits Virtual Data Optimizer

#### LVM

- create a partition, from `parted` use `set n lvm on`
- use `pvcreate /dev/sdb1` to create the physical volume
- use  `vgcreate vgdata /dev/sdb1` to  create the volume group
- use `lvcreate -n lvdata -L 1G vgdata` to create the logical volume
- use `mkfs /dev/vgdata/lvdata` to create a file system
- put in `/etc/fstab` to mount it persistently

 #### VDO

 - ensure that underlying block devices are > 4GiB
 - `yum install vdo kmod-kvdo`
 - `vdo create --name=vdo1 --device=/dev/nvme0np2 --vdoLogicalSize=1T`
 - `mkfs.xfs -K /dev/mapper/vdo1`
 - `udevadm settle` will wait for the system to register the new device name
 - in `/etc/fstab`, include the `x-systemd.requires=vdo.service` mount option
 - monitor using `vdostats --human-readable`


 #### LUKS Encrypted Volumes

 - use `parted` to create a partition
 - `cryptsetup luksFormat` will format the LUKS device
 - `cryptsetup luksOpen` will open it and create a device mapper name
 - mount the resulting device mapper device
 - to automate the `cryptsetup luksOpen`, use `/etc/crypttab`
 - to automate mounting the volume, use `/etc/fstab`

 ## Linux kernel

 #### Kerne Modules

 - Linux drivers are implemented as kernel modules
 - most kernel modules are loaded automatically through `initramfs` or `systemd-udevd`
 - use `modprobe` to manually load kernel module
 - use `lsmod` to list currently loaded kernel modules

 ### `modprobe`

 - use `modprobe` to load a kernel module and all its dependencies
 - use `modprobe -r ` to unload
 - `modinfo` can show module parameters
 - to laod, specify kernel module parameters,edit `/etc/modprobe.conf` or the files in `/etc/modprobe.d`
 

 ### tune kernel

 - `/proc` is a file system that provides access to kernel information
    - PID dir
    - status files
    - tunables in `/proc/sys`
- use `echo` to write a value to any file in `/proc/sys` to change kernel performance parameters
- write the parameters to `/etc/sysctl.conf` to make them persistent
- use `sysctl -a` to show a list of all current settings

### update kerne

- Linux kernel are not technically updated, a new kernel is installed beside the old kernel
- this allows administrators to boot the old kernel in case anything goes wrong
- use either `yum update kernel` or `yum install kernel` to update the kernel

### Boot procedure

 #### modify Grub2 Runtime parameters

 - from the grub2 boot menu, press `e` to edit runtime boot options
 - press `c` to enter the grub2 command mode
    - from command mode, type `help` for an overview of available options
- to edit persitent Grub2 parameters, edit the configuration file in `/etc/default/grub`
- after writing changes, compile changes to `grub.cfg`
    - `grub2-mkconfig -o /boot/grub2/grub.cfg`
    - `grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg`

### system target

- a systemd target is a group of unit files
- some targets are `isolatable`, which means that they define the final state a system is starting in 
    - emergency.target
    - rescue,target
    - multi-user.target
    - graphical.target
- when enableing a unit , it is added to a specific target

#### manage the default target

- use `systemctl get-default` to see the current default target
- use `systemctl set-default` to set a new default target 

#### Booting the specific target

- on the grup 2 boot promp, use `systemctl.unit=xxx.target` to boot into a specific target
- to change between targets on a running system, use `systemctl isolate xxx.target`

## Essential Troubleshooting Module

### Troubleshooting Module

1. GRUB -> `menu: kernel arguments` 
2. kernel initramfs -> `rd.break`
3. systemd -> `init=/bin/bash`
4. base os - > `systemd.unit=emergency.target`
5. service -> `rescure.target`

### change `root` passwd

- enter Grub menu while booting
- find the line that loads the linux kernel and add `rd.break` to end of the line
- `mount -o remount,rw /sysroot`
- `chroot /sysroot`
- `echo secret | passwd --stdin root`
- `touch /.autorelabel` for Selinux

### Troubleshooting Filesystem issue

- real corruption does occur, but not often, and is automatically fixed
- problems occur when making typo's in `/etc/fstab`
- to fix : if necessary, remount filesystem in read/write state and edit `/etc/fstab`
- fragmentation can be an issue, differenct tools exist to fix 
    - `xfs_fsr` is the XFS File System Reorganizer, it optimizes XFS file systems
    - `e4defrag` can be used to defragment Ext4

### Troubleshooting Network issue

- Wrong subnet mask
- Wrong router
- DNS not working

### Troubleshooting Performance Problems

- troubleshooting performance is an art on its own
- focus on the four key area's of performance
    - memory
    - CPU load
    - disk load
    - network

### software issue

- dependency problems in RPM's
    - should not occur when using repositories
- library problems
    - run `idconfig` to update the library cache

### Troubleshooting Memery issue

 # Managing Network Service

## SSH

- `ssh-keygen` create a public/private key pair for the current user
    - setting a passphrase for the private key makes it more secure, but less convenient
- `ssh-cpoy-id` copies the public key over to the target server
- `ssh-agent /bin/bash` allocates space in the bash shell to cache the private key passphrase
- `ssh-add` adds the current passphrase to the cache 

## Apache Configuration

- Apache (httpd) is a leading web server on Linux
- Nginx is the other leading web server
- the main httpd configuration file is `/etc/httpd/conf/httpd.conf`
- additional snap-in file can be stored in `/etc/httpd/conf.d/`
- the default DocumentRoot is `/var/www/htdocs`
- Apache looks for a file with the name `index.html` in this directory

## Managing SELinux

 - Linux security is built on UNIX security
 - UNIX security consists of different solutions that were never developed with current IT security needs in mind
 - Most of these solutions focuse on a part of the operating system
 - SELinux provides a complete and mandatory security solution
 - The principle, "unknown" services will always need addtional configuration to enable them in an environment where SELinux is enabled

 ### SELinux States

 - `getenforce` will show the current state
 - `setenforce` toggles between Enforcing and Permissive
 - Edit `/etc/sysconfig/selinux` to manage the default state of SELinux
 - Never set to disabled if this is meant as a temporary measure only!

 ### Context Labels

 - Every objects is labeled with a context label
    - `user` : user specific context
    - `role` : role specific context
    - `type` : flags which type of operation is allowed on this object
- Many commands suppoert a `-Z` option to show current context information
- Context types are used in the rules in the policy to define witch source object has access to which target object

### File Context Labels

- use `semanage-fcontext` to set  the file context label
    - this will write the context to the SELinux Policy
- to enforce the policy setting on the file system, use `restorecon`
- alternatively, use `touch /.autorelabel` to relabel all filles to the context that is specified in the polict

### SELinux Log

-  SELinux uses `auditd` to write log messages to the audit log
- Messages in the audit log may be hard to interpret
- Ensure that `sealert` is available, it interprets messages from the audit log, applies SELinux AI, and writes meaningful messages to `/var/log/messages`
- Run the `sealert` command, including the UUID message to get advice on how to troubleshoot specific issues

## Firewalled Network

- Service : the main component, contains one or more  ports as well as optional kernel modules that should be loaded
- Zone : a default configuration to which network cards can be assigned to apply specific settings
- Ports : optional elements to allow access to specific ports
- Addtional components are available as well, but not frequently used in a base firewall configuration

### `firewall-cmd`

- the `firewall-cmd` command is used to write firewall configuration
- use the option `--permanent` to write to persistent(but not to runtime)
- without `--permanent` the rule is written to runtime (but not to persistent). Need to restart 

## Automating Installations

- Vagrant is used for automatic deployment of virtual machines
- Cloud-init and other templates can be used in cloud environments
- Kickstart can be used with a PXE-boot server to provide instructions for automatic installations of RHEL

A kickstart file contains all installation instructions to set up  a RHEL instance. It can be used to easily reproduce installations.

### work with kickstart file

- after installation, a file anaconda-ks.cfg is created to the root user home directory
- edit this file manually to make ayn changes that are required

## Configure time service

- `date` `timedatectl` `tzselect`

## Remote File System

- run the `nfs-service` service
- create a directory you want to share : `/data`
- edit `/etc/exports` to contain the following line
    - `/data *(rw,no_root_squash)`

### mounting NFS shares

- use `showmount -e nfs-server` to show exports
- use `mount nfsserver:/share /mnt` to mount
- while mounting through `/etc/fstab`, include the `_netdev` mount option

### configure a Base Samba Server

- install the Samba server package
- create a directory to share 
- create a local Linux user
- Set Linux Permissions
- use `smbpasswd -a` to add a Samba user account
- enable the share in `/etc/samba/samba.conf`
- use `systemctl start smb` to start the service
- use `firewall-cmd --add-service samba --permanent; firewall-cmd --reload` to open the firewall

### mounting Samba Shares

- install the cifs-utils and samba-client RPM packages
- use `smbclient -L //sambahost` to discover shares
- use `mount -o username=sambauser //sambaserver/share /somewhere` to mount the share
- make mount persistent through `/etc/fstab`, using the `_netdev` , `username=` and `passwd=` mount options

### Automount

- in `/etc/auto.master` you will identify the directory that automaount should manage, and the file that is used for additional mount information
    - `/data` `/etc/auto.data`
- in `/etc/auto.data` you will identify the subdirectory on which to mount, and what to mount excactly
    - `files -rw nfsserver:/data/files`
- ensure the autofs service is started:
    - `systemctl enable --now autofs`
    