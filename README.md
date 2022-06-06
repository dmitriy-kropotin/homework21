# homework21

1. Для выполнения домашней работы добавляю пользователей

```
[root@pam21 ~]# useradd day
[root@pam21 ~]# useradd night
[root@pam21 ~]# useradd friday
```

2. Устанавлвиаю пароли и включаю вход по ssh

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

3. Добавляю в  файл `/etc/security/time.conf` следующие параметры авторизации по времени для созданных пользователей

```
[root@pam21 ~]# tail -n5 /etc/security/time.conf
#
*;*;day;Al0800-2000
*;*;night;!Al0800-2000
*;*;friday;Fr0000-2400
```

4. Включаю модуль авторизации по времени в файле `/etc/pam.d/sshd ` строчкой `account    required     pam_time.so`

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

5. Мое текущее время 

```
[tesla@rocky8 homework21]$ date
Mon Jun  6 15:16:57 MSK 2022
```

6. Проверяю. Под day пускает

```
[tesla@rocky8 homework21]$ ssh day@localhost -p 2200
day@localhost's password:
[day@pam21 ~]$
```

7. Под night не пускает

```
[tesla@rocky8 homework21]$ ssh night@localhost -p 2200
night@localhost's password:
Connection closed by 127.0.0.1 port 2200
```

8. Под friday не пускает. Модуль работае!

```
[tesla@rocky8 homework21]$ ssh friday@localhost -p 2200
friday@localhost's password:
[friday@pam21 ~]$ logout
Connection to localhost closed.
```

9. Включаю модуль `pam_exec.so` для запуска скрипта при авторизации 

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

10. Содержимое скрипта

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

11. Добавлюя права на запуск скрипта

```
[root@pam21 ~]# chmod +x /usr/local/bin/test_login.sh
[root@pam21 ~]# ls -la /usr/local/bin/test_login.sh
-rwxr-xr-x. 1 root root 444 Jun  6 13:24 /usr/local/bin/test_login.sh
```

12. Пробую под `friday` и `night`. Не пускает

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

13. Пробую под `day`. Пускает! Скрипт работае. 

```
[tesla@rocky8 homework21]$ ssh day@localhost -p 2200
day@localhost's password:
Last login: Mon Jun  6 12:15:01 2022 from 10.0.2.2
[day@pam21 ~]$
```

14. Теперь буду тестировать модуль `pam_script`, устанавлвиаю его. 

```
[root@pam21 ~]# dnf install  pam_script
Last metadata expiration check: 1:06:38 ago on Mon 06 Jun 2022 12:25:28 PM UTC.
Dependencies resolved.
========================================================================================================================================================
 Package                               Architecture                      Version                                  Repository                       Size
========================================================================================================================================================
Installing:
 pam_script                            x86_64                            1.1.9-7.el8                              epel                             34 k
....
Installed:
  pam_script-1.1.9-7.el8.x86_64

Complete!
```

15. Включаю его строчкой `account    required     pam-script.so /usr/local/bin/test_login.sh` 

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

16.Проверяю, 

```
[tesla@rocky8 homework21]$ ssh friday@localhost -p 2200
friday@localhost's password:
Connection closed by 127.0.0.1 port 2200
[tesla@rocky8 homework21]$ ssh day@localhost -p 2200
day@localhost's password:
Connection closed by 127.0.0.1 port 2200
```

17. И не работает( под `day` не пускает. Судя по документации файл скрипта для секции `account` должен называется `pam_script_acct`. Переименовываю и через параметр `dir=/usr/local/bin/` указываю директорию со скриптами.

```
[root@pam21 log]# cp /usr/local/bin/test_login.sh /usr/local/bin/pam_script_acct
[root@pam21 log]# ls -la /usr/local/bin/
total 8
drwxr-xr-x.  2 root root  50 Jun  6 13:52 .
drwxr-xr-x. 12 root root 131 May 21 20:03 ..
-rwxr-xr-x.  1 root root 444 Jun  6 13:52 pam_script_acct
-rwxr-xr-x.  1 root root 444 Jun  6 13:24 test_login.sh
```

```
[root@pam21 log]# cat /etc/pam.d/sshd
#%PAM-1.0
auth       substack     password-auth
auth       include      postlogin
account    required     pam_sepermit.so
account    required     pam_nologin.so
account    required     pam_script.so dir=/usr/local/bin/
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

18. Проверяю, все работает, под `day` пускает, под `night` и `friday` нет.

```
[tesla@rocky8 homework21]$ ssh friday@localhost -p 2200
friday@localhost's password:
Connection closed by 127.0.0.1 port 2200
[tesla@rocky8 homework21]$ ssh day@localhost -p 2200
day@localhost's password:
Last login: Mon Jun  6 13:49:22 2022 from 10.0.2.2
[day@pam21 ~]$
```

19. Теперь пользоватею day назначу специфические права, в частности право на открытие порта. У обычного пользователя таких прав нет. 

```
[day@pam21 ~]$ ncat -l -p 80
Ncat: bind to :::80: Permission denied. QUITTING.
```

20. Подключаю модуль `auth       required     pam_cap.so`

```
[root@pam21 log]# cat /etc/pam.d/sshd
#%PAM-1.0
auth       substack     password-auth
auth       include      postlogin
auth       required     pam_cap.so
account    required     pam_sepermit.so
account    required     pam_nologin.so
#account    required    pam_script.so dir=/usr/local/bin/
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

21. Добавляю право пользователю `day`

```
[root@pam21 log]# cat /etc/security/capability.conf
cap_net_bind_service day
```

22. Выдаю права программе

```
[root@pam21 log]# setcap cap_net_bind_service=ei /usr/bin/ncat
```

23. Захожу под `day` и проверяю выданное право

```
[day@pam21 ~]$ capsh --print
Current: cap_net_bind_service=i
```

24. Открываю порт, и отправляю в него `echo "Make Linux great again!" > /dev/tcp/127.0.0.7/80`. Все работает!

```
[day@pam21 ~]$ ncat -l -p 80
Make Linux great again!
```

25. Выдам права `root` пользователю `day` без запроса пароля. Для этого создам файл `/etc/sudoers.d/day`

```
[root@pam21 log]# cat /etc/sudoers.d/day
day ALL=(ALL) NOPASSWD: ALL
```

26. Проверяю, работает!

```
[tesla@rocky8 homework21]$ ssh day@localhost -p 2200
day@localhost's password:
Last login: Mon Jun  6 14:15:13 2022 from 10.0.2.2
[day@pam21 ~]$ sudo su -l
Last login: Mon Jun  6 11:19:44 UTC 2022 on pts/0
[root@pam21 ~]#
```
