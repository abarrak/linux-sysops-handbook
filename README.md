# Linux SysOps Handbook

A study notes book for the common knoweldge and tasks of a Linux system admin.

## Table of Content


## 1. Processes


## 2. User Management

The users and groups are managed in `/etc/passwd` and `/etc/group` files.

The commands to manage a user are as follows:
- `useradd`
- `usermod`
- `userdel`

And for groups are 

To lock or unlock a user account, us the `-L`, `-U` options respectively.

```shell
$ usermod -L <username>
$ usermod -U <username>
````

## 3. Shell Tips and Tricks 


## 4. File Permissions

A file permissions are considered in three dimensions: the owner user, the owner's group, and rest of other users. Showing the permisison of files and directories can be using `ls -l`, `ls -ld` respectively.

The basic permission types are: read (r), write (w), and execute (x) on both folders and files:

```shell
$ ls -l
-rw-r--r--  1 abdullah  staff  35149 Jan 30 17:20 LICENSE
```


## 5. Crons and Background Services


## 6. Linxu Distros


## 7. Logs, Monitroing, and Troubleshooting


## 8. Network Essentials


## 9. System Updates and Patching


## 10. Additional Resources & Final Notes


## License

[GNU General Public License v3.0](https://github.com/abarrak/linux-sysops-handbook/blob/main/LICENSE).
