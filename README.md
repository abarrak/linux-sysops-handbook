# Linux SysOps Handbook

A study notes book for the common knoweldge and tasks of a Linux system admin.

## Table of Content

1. [Processes](#1-processes)
2. [User Management](#2-user-management)
3. [Shell Tips and Tricks](#3-shell-tips-and-tricks)
4. [File Permissions](#4-file-permissions)
5. [Crons and Background Services](#5-crons-and-background-services)
6. [Linxu Distros](#6-linxu-distros)
7. [Logs, Monitroing, and Troubleshooting](#7-logs-monitroing-and-troubleshooting)
8. [Network Essentials](#8-network-essentials)
9. [System Updates and Patching](#9-system-updates-and-patching)
10. [Additional Resources & Final Notes](#10-additional-resources--final-notes)

## 1. Processes

List the current active process with their statuses, numbers, resource usage, etc. using the command `ps`.

```shell
$ ps -auxc
```

Quoting man's page documentaiton on `ps`: "A different set of processes can be selected for display by using any combination of the `-a, -G, -g, -p, -T, -t, -U, and -u` options.  If more than one of these options are given, then ps will select all processes which are matched by at least one of the given options".

The daemon `systemd` process starts during boot time, and remains active until the shutdown. It's the parent process for all other process in the system.

Each process contains several main parts, such as: PID, state, virtual space address (memory), threads, network and file descriptors, scheduler information, and links. Processes are controlled and respond to signals. The states that a process can transistion among are depicted below:

<img src="https://github.com/abarrak/linux-sysops-handbook/blob/main/images/process-states.png?raw=true" width="700px" height="450px" />

To observe the states and other information of the processes interatively, use the `top` command.


## 2. User Management

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

To change a user passowrd, use the command `passwd` interactively. Additionally `change` command sets the password policy in the system.


Use the command `su - <username>` to switch to the specified user. which will promote for her password. Running the command without username will switch to
the root user. To avoid cases where password is not availabale, use `sudo` to switch accounts using current user password only and according to rules in `/etc/sudoers` directory. Use `sudo -i` to gain an interactive root shell.


To run executables as background process (job), append an ampersand to it:

```shell
$ echo "Hi .. looping:" | sleep 10000 | echo "done." &
````

To view the current jobs, and their details run `job`, `ps j` commands respectively.

To bring back a job in the foreground in the current session, and send it back use the followin:

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

Use `killall` to operate on multiple processes using their executable name. Use `pkill` for filering with more options.

```shell
$ killall -15 nginx
$ pkill -U tester
```


## 3. Shell Tips and Tricks 


## 4. File Permissions

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

## 5. Background Services and Crons

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


## 6. Linxu Distros


## 7. Logs, Monitroing, and Troubleshooting

You can moitor the system's resources usage, uptime, and sessions' load leverages over time as follows:

```shell
$ top
$ uptime
$ w
```

Use `lscpu` to see the system's CPU in use and other details.


## 8. Network Essentials


## 9. System Updates and Patching


## 10. Additional Resources & Final Notes

Use the `man` command to lookup the manual information on commands or topics in the system.

Additionally, the `info` command is the GNU documentation tool and provide more detailed materials.

Both provide shortcuts, navigation, and searching capablities (e.g. `man -K <keyword` to search across manual).

**Recommened Reading list:**

1. [How Linux Worksm What Every Superuser Should Know, Brian Ward, _2nd Edition, No Starch Press_.](https://www.amazon.com/How-Linux-Works-2nd-Superuser/dp/1593275676).
2. [Linux Command Line and Shell Scripting Bible, R. Blum and C. Bresnahan, _3rd Edition, Wiley_.](https://www.amazon.com/Command-Scripting-Christine-Bresnahan-2015-01-20/dp/B01JNWWSZA)
3. [Linux Bible, Christopher Negus, _9th Edition, Wiley_.](https://www.amazon.com/Linux-Bible-Christopher-Negus/dp/1119578884/)
4. [9 to 5 Linux Blog.](https://9to5linux.com/)
5. [nixCraft.](https://www.cyberciti.biz/)

## License

[GNU General Public License v3.0](https://github.com/abarrak/linux-sysops-handbook/blob/main/LICENSE).
