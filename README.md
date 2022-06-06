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
*;*;day;Al0800-2000
*;*;night;!Al0800-2000
*;*;friday;Fr0000-2400

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



```
[root@pam21 ~]# cat /etc/pam.d/sshd
#%PAM-1.0
auth       substack     password-auth
auth       include      postlogin
account    required     pam_sepermit.so
account    required     pam_nologin.so
account    required     pam_exec.so /usr/local/bin/test_login.sh
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
[root@pam21 ~]# cat /usr/local/bin/test_login.sh
#!/bin/bash

if [ $PAM_USER = "friday" ]; then
  if [ $(date +%a) = "Fri" ]; then
      exit 0
    else
      exit 1
  fi
fi

hour=$(date +%H)

is_day_hours=$(($(test $hour -ge 8; echo $?)+$(test $hour -lt 20; echo $?)))

if [ $PAM_USER = "day" ]; then
  if [ $is_day_hours -eq 0 ]; then
      exit 0
    else
      exit 1
  fi
fi

if [ $PAM_USER = "night" ]; then
  if [ $is_day_hours -eq 1 ]; then
      exit 0
    else
      exit 1
  fi
fi
```

```
[root@pam21 ~]# chmod +x /usr/local/bin/test_login.sh
[root@pam21 ~]# ls -la /usr/local/bin/test_login.sh
-rwxr-xr-x. 1 root root 444 Jun  6 13:24 /usr/local/bin/test_login.sh
```

```
[tesla@rocky8 homework21]$ ssh friday@localhost -p 2200
friday@localhost's password:
/usr/local/bin/test_login.sh failed: exit code 1
Connection closed by 127.0.0.1 port 2200
```

```
[tesla@rocky8 homework21]$ ssh night@localhost -p 2200
night@localhost's password:
/usr/local/bin/test_login.sh failed: exit code 1
Connection closed by 127.0.0.1 port 2200
```

```
[tesla@rocky8 homework21]$ ssh day@localhost -p 2200
day@localhost's password:
Last login: Mon Jun  6 12:15:01 2022 from 10.0.2.2
[day@pam21 ~]$
```

```
[root@pam21 ~]# dnf install  pam_script
Last metadata expiration check: 1:06:38 ago on Mon 06 Jun 2022 12:25:28 PM UTC.
Dependencies resolved.
========================================================================================================================================================
 Package                               Architecture                      Version                                  Repository                       Size
========================================================================================================================================================
Installing:
 pam_script                            x86_64                            1.1.9-7.el8                              epel                             34 k

Transaction Summary
========================================================================================================================================================
Install  1 Package

Total download size: 34 k
Installed size: 63 k
Is this ok [y/N]: y
Downloading Packages:
pam_script-1.1.9-7.el8.x86_64.rpm                                                                                       170 kB/s |  34 kB     00:00
--------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                    28 kB/s |  34 kB     00:01
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                1/1
  Installing       : pam_script-1.1.9-7.el8.x86_64                                                                                                  1/1
  Running scriptlet: pam_script-1.1.9-7.el8.x86_64                                                                                                  1/1
  Verifying        : pam_script-1.1.9-7.el8.x86_64                                                                                                  1/1

Installed:
  pam_script-1.1.9-7.el8.x86_64

Complete!
```

```
[root@pam21 ~]# cat /etc/pam.d/sshd
#%PAM-1.0
auth       substack     password-auth
auth       include      postlogin
account    required     pam_sepermit.so
account    required     pam_nologin.so
account    required     pam-script.so /usr/local/bin/test_login.sh
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
[tesla@rocky8 homework21]$ ssh friday@localhost -p 2200
friday@localhost's password:
Connection closed by 127.0.0.1 port 2200
```



