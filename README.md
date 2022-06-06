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
