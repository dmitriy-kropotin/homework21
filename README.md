# homework21

```
[root@pam21 ~]# useradd day
[root@pam21 ~]# useradd night
[root@pam21 ~]# useradd friday
```


```
[root@pam21 ~]# echo "otus1234"|sudo passwd --stdin day
Changing password for user day.
passwd: all authentication tokens updated successfully.
[root@pam21 ~]# echo "otus1234"|sudo passwd --stdin night
Changing password for user night.
passwd: all authentication tokens updated successfully.
[root@pam21 ~]# echo "otus1234"|sudo passwd --stdin friday
Changing password for user friday.
passwd: all authentication tokens updated successfully.

[root@pam21 ~]# sed -i 's/^PasswordAuthentication.*$/PasswordAuthentication yes/' /etc/ssh/sshd_config && systemctl restart sshd.service
[root@pam21 ~]#
```

```
[root@pam21 ~]# tail -n5 /etc/security/time.conf
#
#
*;*;day;Al0800-2000
*;*;night;!Al0800-2000
*;*;friday;Fr
```

```
[root@pam21 ~]# cat /etc/pam.d/sshd
#%PAM-1.0
auth       substack     password-auth
auth       include      postlogin
account    required     pam_sepermit.so
account    required     pam_nologin.so
account    required     pam_time.so
account    include      password-auth
password   include      password-auth
# pam_selinux.so close should be the first session rule
session    required     pam_selinux.so close
session    required     pam_loginuid.so
# pam_selinux.so open should only be followed by sessions to be executed in the user context
session    required     pam_selinux.so open env_params
session    required     pam_namespace.so
session    optional     pam_keyinit.so force revoke
session    optional     pam_motd.so
session    include      password-auth
session    include      postlogin
```

```
[tesla@rocky8 homework21]$ ssh day@localhost -p 2200
day@localhost's password:
[day@pam21 ~]$
```

```
[tesla@rocky8 homework21]$ ssh night@localhost -p 2200
night@localhost's password:
Connection closed by 127.0.0.1 port 2200
```

```
[tesla@rocky8 homework21]$ date
Mon Jun  6 15:16:57 MSK 2022
```
