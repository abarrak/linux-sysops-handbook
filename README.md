# Linux SysOps Handbook

A study notes book for the common knoweldge and tasks of a Linux system admin.

## Table of Content


## 1. Processes


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


## 3. Shell Tips and Tricks 


## 4. File Permissions

A file permissions are considered in three dimensions: the owner user, the owner's group, and rest of other users. Showing the permisison of files and directories can be using `ls -l`, `ls -ld` respectively.

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


## 5. Crons and Background Services


## 6. Linxu Distros


## 7. Logs, Monitroing, and Troubleshooting


## 8. Network Essentials


## 9. System Updates and Patching


## 10. Additional Resources & Final Notes

Use the `man` command to lookup the manual information on commands or topics in the system.

Additionally, the `info` command is the GNU documentation tool and provide more detailed materials.

Both provide several shortcuts, navigation, and searching capablities (e.g. `man -K <keyword` to search across the manual).

**Recommened Reading list:**
1.
2.
3. 
4. 
5.

## License

[GNU General Public License v3.0](https://github.com/abarrak/linux-sysops-handbook/blob/main/LICENSE).
