# Linux SysOps Handbook

A study notes book for the common knowledge and tasks of a Linux system admin.

[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
<a href="https://abdullah-barrak.gitbook.io/linux-sysops-handbook" alt="Gitbook link"><img src="https://img.shields.io/badge/gitbook-link-success" /></a>
<a href="https://github.com/abarrak/linux-sysops-handbook" alt="Github link"><img src="https://img.shields.io/badge/github-link-important" /></a>
![GitHub contributors](https://img.shields.io/github/contributors/abarrak/linux-sysops-handbook)

## Table of Content

- [1. Processes](#processes)
- [2. User Management](#user-management)
- [3. Shell Tips and Tricks](#shell-tips-and-tricks)
- [4. File Permissions](#file-permissions)
- [5. Background Services and Crons](#background-services-and-crons)
- [6. Linux Distros](#linux-distros)
- [7. Logs, Monitoring, and Troubleshooting](#logs-monitoring-and-troubleshooting)
- [8. Network Essentials](#network-essentials)
- [9. System Updates and Patching](#system-updates-and-patching)
- [10. Storage](#storage)
- [11. Notes & Additional Resources](#notes-and-additional-resources)

## Processes

List the current active process with their statuses, numbers, resource usage, etc. using the command `ps`.

```shell
$ ps auxc
```

Quoting man's page documentaiton on `ps`: "A different set of processes can be selected for display by using any combination of the `-a, -G, -g, -p, -T, -t, -U, and -u` options.  If more than one of these options are given, then ps will select all processes which are matched by at least one of the given options".

The daemon `systemd` process starts during boot time, and remains active until the shutdown. It's the parent process for all other process in the system.

Each process contains several main parts, such as: PID, state, virtual space address (memory), threads, network and file descriptors, scheduler information, and links. Processes are controlled and respond to signals. The states that a process can transistion among are depicted below:

<img src="https://github.com/abarrak/linux-sysops-handbook/blob/main/images/process-states.png?raw=true" width="700px" />

To observe the states and other information of the processes interatively, use the `top` command.

To run executables as background process (job), append an ampersand to it:

```shell
$ echo "Hi .. looping:" | sleep 10000 | echo "done." &
```

To view the current jobs, and their details run `job`, `ps j` commands respectively.

To bring back a job in the foreground in the current session, and send it back use the following:

```shell
$ fg %<job-no>
$ ctrl+z
$ bg %<job-no>
```

Use the command `kill -l` to see the available signals to send to processes, like interrupt, terminate, resume, etc.

```shell
$ kill -l
$ kil -9 5921
$ kill -SIGTERM 6152
```

Use `killall` to operate on multiple processes using their executable name. Use `pkill` for filtering with more options.

```shell
$ killall -15 nginx
$ pkill -U tester
```

## User Management

The users and groups are managed in `/etc/passwd` and `/etc/group` files.

```shell
$ tail /etc/passwd
$ tail /etc/group
$ tail /etc/shadow
```

The commands to manage a user are as follows:
- `useradd`
- `usermod`
- `userdel`

And for groups:
- `groupadd`
- `groupmod`
- `groupdel`

Each user in the system is associated with unique user id `uid`, and each group is associated with `gid`.

```shell
$ id abdullah
```

Use flags `-g` and `-aG` for users to replace group or append group, respectively:

```shell
$ sudo usermod -G admins abdullah
$ sudo usermod -aG staff abdullah
```

To lock or unlock a user account, us the `-L`, `-U` options respectively.

```shell
$ usermod -L <username>
$ usermod -U <username>
```

To restrict service user accounts (e.g. accounts for web servers), the shell can be set to `nologin`:

```shell
$ usermod -s /sbin/nologin nginx_usr1
```

To change a user password, use the command `passwd` interactively. Additionally `change` command sets the password policy in the system.


Use the command `su - <username>` to switch to the specified user. which will promote for her password. Running the command without username will switch to
the root user. To avoid cases where password is not available, use `sudo` to switch accounts using current user password only and according to rules in `/etc/sudoers` directory. Use `sudo -i` to gain an interactive root shell.


## Shell Tips and Tricks 


Getting used to [bash language and its fundamentals](https://learnxinyminutes.com/docs/bash/) like conditions, looping, functions, etc. is recommended.

The popular files and text processing and manipulation utilites are important to master, such as:

- `cat`
- `cp`
- `rm`
- `mkdir`
- `rmdir`
- `touch`
- `less`
- `more`
- `head`
- `tail`
- `grep`
- `find`
- `locate`
- `wc`
- `sed`

Use the command `date` to print the current date and time or others in the past and future:

```shell
$ date +%x
```

The standard terminal channels in Linux are 3: `stdin`, `stdout`, and `stderr` where the first is for input stream and the latters for output and error streams. 

By default the successful command results are outputted to `stdout` (equivalent to `>`). You can explicity redirect to `stdout` or `stderr` as follows:

```shell
$ echo "hi there!" 1> error_log.txt
$ cat ~/incorrect-path 2> error_log.txt
# To both:
$ (echo "hi" && cat ~/wrong) >> log.txt 2>&1
```

To discard output stream, redirect it to the special directory `/dev/null`.

The standard input can be captured via redirection or file pipes:

```shell
$ cat <<EOF
This is coming from the stdin
EOF

$ cat LICENSE | wc -l
```

The `ssh` command used to connect to servers in secure manner using OpenSSH library using public key cryptography. The configuration and known hosts are kept under `/etc/ssh` system-wide or in `~/.ssh/` in current user's home directory. On the other hand `scp` is used for secure copy on secure shell fashion.

The following list of commands are used to generate and manage ssh keys between client and server:

1. `ssh-keygen`.
2. `ssh-agent`.
3. `ssh-copy-id`.
4. `ssh-add`.


## File Permissions

A file permissions are considered in three dimensions: the owner user, the owner's group, and rest of other users. 

Showing the permisison of files and directories can be using `ls -l`, `ls -ld` respectively.

The basic permission types are: read (r), write (w), and execute (x) on both folders and files:

```shell
$ ls -l
-rw-r--r--  1 abdullah  staff  35149 Jan 30 17:20 LICENSE
```

Setting the files and folders permission is done by `chmod` command and can be using symbols or digits. 

The symbols/letter way is made for `u`, `g`, `o`, or `a` basis for the user, group, others, or all. Whereas, the digits are written for all at once in sequence for user, group, and others. Examples are below for both cases:

```shell
# Use + to add, - to remove, and = to reset.

# adding execute permission to user
$ chmod u+x my-file.txt
# setting read, execute to all on a folder and its content
$ chmod -R a=rX my-folder

$ chmod 740 special.txt
$ chmod -R 444 read-only-files/
```

`chown` is used to change the ownership of folder/files to users or groups respectively. `chgrp` is a shortcut to group change only. The root or the owner are only people can change ownership and in the latter, she needs to be part of the new target group before the change.

```shell
$ chown sarah file-10.txt
$ chown sarah:staff file-12.txt

$ chown :admins server_log.txt
$ chgrp operators server_log.txt
```

Lastly, a fourth dimension at the start can be added to represent the special permissions of `suid s`, `sgid s`, and `sticky t` which control executable nature of files to be of owner users, and groups regardless of the current user. The last is to restrict deletion for only the root and owner always.

```shell
$ chmod a+t protected-folder/
$ chmod -R 1444 read-only-protected/
```

Finally, use `pstree` and `pgrep` to view process parent/child tree and search for processes by pattern.

```shell
$ psgrep -u abdullah -l
```

## Background Services and Crons

`systemctl` is the command used to list, manage, and check background processes or so called `daemons`.

To list the available categories of daemons, run:

```shell
$ systemctl -t help
```

There are 3 types of daemons: 1. services, 2. sockets, 3. paths. Use the following to see the system's processes in each:

```shell
$ systemctl
$ systemctl list-units --type=service
$ systemctl list-units --type=socket --state=LOAD
$ systemctl list-units --type=path --all
$ systemctl list-unit-files
```

The states `enabled` and `disabled` indicate wether a service is lanuched on startup or not. The subcommands `enable` and `disable` can be used to control this aspect.

To view the status of a daemon use the `status` command or its state shortcuts:

```shell
$ systemctl status kubelet
$ systemctl is-active dockerd
$ systemctl is-enabled sshd.service
```

Use the subcommands `start`, `stop`, `restart`, and `reload`, `reload-or-restart` to control daemons.

Additionally, use the following to list a daemon dependencies:

```shell
$ systemctl list-dependencies nginx.service
```

Finally, to resolve conflicting services making them unavailable, the `mask` and `unmask` commands can be used to point a deamons config to `dev/null` then back to normal respectively.


The cron daemon `crond` is responsible for managing the user's and system's scheduled jobs. Use the command `crontab` to manage jobs and their files in the user account or in the system wide `/etc/crontab`, `/etc/cron.d/` locations.

```shell
$ sudo crontab -l
$ sudo crontab -e
$ vim /etc/cron.d/my-backup
```

The syntax of crontab entries  is captured by the diagram below. Use the [following tool to quick assistance.](https://crontab.guru/)

<img src="https://github.com/abarrak/linux-sysops-handbook/blob/main/images/crontabs.jpg?raw=true" />

An example of a cron entry that runs backup command, every day at 5:00 AM:

```shell
0 5 * * * /usr/bin/daily-backup
```

## Linux Distros

In 1991, Linux kernel was introduced by Linus Torvalds, and combined with GNU project, which was previously created in 1983-1984 as open source OS programs and components. This formed what we call today Linux distribution, a Unix-like operating system.

Today the Linux operating system is supported on most hardware platforms.  [Linux works on almost every architecture from i386 to SPARC](https://www.linuxtrainingacademy.com/linux-distribution-intro/). Linux can be found on almost every type of device today, from watches, televisions, mobile phones, servers, desktops, and even vending machines.

One of the major distinction between Linux distributions is the package management part and how software is installed and managed. There are multiple package formats, and the most common ones are Debian (deb), RedHat Package Manager (RPM).

<img src="https://github.com/abarrak/linux-sysops-handbook/blob/main/images/distros.jpg?raw=true" width="700px" />

Here's a listing for the common Debian based distros:

- Debian.
- Ubuntu.
- Linux Mint.
- Kali Linux.

And here's for RPM based distros:

- Fedora.
- RedHat Enterprise Linux (RHEL).
- CentOS.
- openSUSE.

<img src="https://github.com/abarrak/linux-sysops-handbook/blob/main/images/timeline.png?raw=true" width="700px" />

## Logs, Monitoring, and Troubleshooting

You can monitor the system's resources usage, uptime, and sessions' load leverages over time as follows:

```shell
$ top
$ uptime
$ w
```

Use `lscpu` to see the system's CPU in use and other details.

The logs of the system events and processes traces are usually kept in `/var/log` directory. There are two categories of persistent logs (`rsyslogs`) and temporary logs (`journald`) that are wiped across boots. Logs include syslog protocol messages, events, security incidents, mailing logs, jobs logs, and other program logs.


As explored in section (3), use `cat`, `head`, `tail` commands to interactively see or follow the logs.

```shell
$ head -n 50 /var/logs/mail.log
$ tail -f /var/logs/mysql.log
```

You can configure the syslog service and manage it as any daemon:

```shell
$ vim /etc/rsyslog.conf
$ systemctl reload rsyslog
```

On the other hand, use `journalctl` to view and follow the system's `journald` log entries, which resides in `run/log/journal`.

```shell
$ journalctl -n 50 -p err 
$ journalctl -f
$ journalctl _PID=6610
```


## Network Essentials

For effective work on the system network configurations and troubleshooting, it is essential to review network/internet protocols (TCP/UDP) and IPv4/IPv6 concepts [(Ref.1)](https://www.ibm.com/cloud/learn/networking-a-complete-guide), [(Ref.2)](https://www.cloudflare.com/learning/network-layer/what-is-a-protocol/).


See the hostname of current machine or set it as below:

```shell
$ hostname
$ hostnamectl set-hostname rhel.n1.apps.com
```

The host name is managed under `/etc/hostname`. 

The host connection is either managed dynamically (`DHCP`) configured in `/etc/resolv.conf` or manually in `/etc/hosts` file.

The `ping` utiltiy helps for connectivity checking:

```shell
$ ping 172.168.9.13
$ ping -c4 github.com
$ ping6 2001:db8:3333:4444:5555:6666:7777:8888
```

To see the network routing table and interfaces, use the following:

```shell
$ ip routes
$ ip -6 route
$ ip help
$ ip show link
```

Use the command `nmap` [for advanced network investigation and security monitor and scan.](https://www.cyberciti.biz/security/nmap-command-examples-tutorials/)

```shell
# Scan a single ip address
$ nmap 192.168.1.1
 
# Scan a host name 
$ nmap -v server1.cyberciti.biz

# View open ports:
$ nmap --open 192.168.2.18

# Trace all pakets:
$ nmap --packet-trace 192.168.1.1
```

`NetworkManager` is the kernel feature [to manage netowrk configurations in Linux](https://en.wikipedia.org/wiki/NetworkManager). `nmcli` is the terminal utility.

```shell
$ nmcli device wifi list
$ nmcli dev status
$ nmcli general hostname centos-8.cluster.internal
$ nmcli con show 
```


## System Updates and Patching

Managing the system packages varies depending on linux distributions, but the essential parts are the same (installation, repositories, package managers, etc.). For Debian based distribtuions, `apt` is the package manager, whereas for Fedora / RHEL, `yum` is used.

Search for some package:

```shell
$ apt search <KEYWORD>
$ yum search <KEYWORD>
```

Install a package:

```shell
$ apt install <NAME>
$ yum install <NAME>
```


Update a package or all packages:

```shell
$ apt upgrade <NAME>
$ yum update <NAME>
```

Remove a package:

```shell
$ apt remove <NAME>
$ yum remove <NAME>
```

Show details on a package:

```shell
$ apt show <NAME>
$ yum info <NAME>
```

List all current packages on the system:

```shell
$ apt list --installed
$ yum list
```

Audit the history of pacakge management actions:

```shell
$ cat less /var/log/apt/history.log | less
$ cat less /var/log/dnf.rpm.log | less
```

And finally, the package source repos can be set up and updated through the following:

```shell
# list current enabled repos
$ yum repolist all
$ apt-cache policy

# manage and add repos in these directories:
$ cat /etc/apt/sources.list /etc/apt/sources.list.d/*
$ cat /etc/yum.repos.d/*
```

## Storage

Linux is formed for a unified file-system consists of all file systems provided by the hardware or virtual storage devices attached to the system. Essentially, everything in linux is a file. It can be viewed as a reversed tree of nested directories starting from the root directory `/`.

<img src="https://github.com/abarrak/linux-sysops-handbook/blob/main/images/linux-file-system.png?raw=true" width="700px" />

Block devices are the mechanism that the kernel detects and identify raw storage devices (HDD, SSD, USBs, ..). [As the name indicates, the kernel interfaces and references them by fixed-size blocks (chunks of spaces)](https://www.digitalocean.com/community/tutorials/an-introduction-to-storage-terminology-and-concepts-in-linux). The block devices are stored in `/dev` directory by the OS, and has letters naming convention such as `/dev/sda`, `/dev/sdb`, `/dev/vda`, and appended numbers in case of partitions `/dev/sda3`. The attachment of the block device into the system is done through mounting it to a directory in the system.

Two operations are essential for using block storages:

**1. Partition:**

  Breaking the disk into reusable smaller units, each treated as own disk. 
  The main partitioning methods are MBR (Master Boot Record) and GPT (GUID Partition Table).

**2. Formatting:**

  Prepeating the device as a file-system to be read and write to. Many file-system formats exists like:

  - `Ext4`.
  - `XFS`.
  - `Btrfs`.
  - `ZFS`.


Additionall, LVM and RAID are another two concepts where the first operate on the opposite of partitioning and group multiple disks as one logical volume. The latter (Redundant Array of Independent Disks) is used to architect more advanced storage setup to ensure higher availablity, redundency, RD, etc.

To see the currently attached file system with mounts and a directory space usage, run `df`/`du` commands:

```shell
$ df -H
$ du -H /home/abdullah
```

The `lsof` command lists all active proccess using the block device.

The permanent mounting process rely on `/etc/fstab` file to determine devices to mount on the boot time.

Use the commands `lsblk` and `monunt` to check and mount file-sytem devices, respectively.


## Notes and Additional Resources

Use the `man` command to lookup the manual information on commands or topics in the system.

Additionally, the `info` command is the GNU documentation tool and provide more detailed materials.

Both provide shortcuts, navigation, and searching capablities (e.g. `man -K <keyword` to search across manual).

### Recommened Reading List

**Books:**

1. [How Linux Works What Every Superuser Should Know, Brian Ward, _2nd Edition, No Starch Press_.](https://www.amazon.com/How-Linux-Works-2nd-Superuser/dp/1593275676).
2. [Linux Command Line and Shell Scripting Bible, R. Blum and C. Bresnahan, _3rd Edition, Wiley_.](https://www.amazon.com/Command-Scripting-Christine-Bresnahan-2015-01-20/dp/B01JNWWSZA)
3. [Linux Bible, Christopher Negus, _9th Edition, Wiley_.](https://www.amazon.com/Linux-Bible-Christopher-Negus/dp/1119578884/)

**Websites & Blogs:**

1. [Digital Occean Knowledge Hub.](https://www.digitalocean.com/community/tags/linux-basics?language=en)
2. [9 to 5 Linux Blog.](https://9to5linux.com/)
3. [nixCraft.](https://www.cyberciti.biz/)


## License

[GNU General Public License v3.0](https://github.com/abarrak/linux-sysops-handbook/blob/main/LICENSE).
