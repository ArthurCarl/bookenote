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