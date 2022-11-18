# NFS
```
OTUS home work NFS server and client
CentOS Linux release 7.8.2003 (Core)
nfss - server
nfsc - client
```

Загружаем виртуальные машины (nfss и nfsc) Vagrant и вводим команду:
```
$ vagrant up
```
Проверяем нашли виртульные машины
```
$ vagrant status
```
```
Current machine states:
nfss                      running (virtualbox)
nfsc                      running (virtualbox)
```
        ***Настраиваем SERVER**
Заходим на сервер для настройки NFS:
```
$ vagrant ssh nfss
```
Получаем рут права:
```
$ sudo -i
```
Ставим NFS утилиты:
```
$ yum install nfs-utils -y
```

Включаем firewall и проверяем, что он работает:
```
$ systemctl enable firewalld --now
```
Output:
```
Created symlink from /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service to /usr/lib/systemd/system/firewalld.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/firewalld.service to /usr/lib/systemd/system/firewalld.service.
```
Разрешаем в firewall доступ к сервисам NFS и перезапускаем сервис:
```
$ firewall-cmd --add-service="nfs3" \
            --add-service="rpc-bind" \
             --add-service="mountd" \
             --permanent
$ firewall-cmd --reload

```
Включаем сервер NFS:
```
$ systemctl enable nfs --now
```
Output:
```
Created symlink from /etc/systemd/system/multi-user.target.wants/nfs-server.service to /usr/lib/systemd/system/nfs-server.service.
Status:
[root@nfss ~]# systemctl status nfs
● nfs-server.service - NFS server and services
   Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; enabled; vendor preset: disabled)
   Active: active (exited) since Fri 2022-11-18 10:33:37 UTC; 1min 1s ago
  Process: 3569 ExecStartPost=/bin/sh -c if systemctl -q is-active gssproxy; then systemctl reload gssproxy ; fi (code=exited, status=0/SUCCESS)
  Process: 3553 ExecStart=/usr/sbin/rpc.nfsd $RPCNFSDARGS (code=exited, status=0/SUCCESS)
  Process: 3552 ExecStartPre=/usr/sbin/exportfs -r (code=exited, status=0/SUCCESS)
 Main PID: 3553 (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/nfs-server.service

Nov 18 10:33:37 nfss systemd[1]: Starting NFS server and services...
Nov 18 10:33:37 nfss systemd[1]: Started NFS server and services.
```
Проверяем наличие слушаемых портов:
```
$ ss -tnplu | grep -E "(2049|20048|111)"
Otput:
udp    UNCONN     0      0         *:20048                 *:*                   users:(("rpc.mountd",pid=3551,fd=7))
udp    UNCONN     0      0         *:111                   *:*                   users:(("rpcbind",pid=339,fd=6))
udp    UNCONN     0      0         *:2049                  *:*
udp    UNCONN     0      0      [::]:20048              [::]:*                   users:(("rpc.mountd",pid=3551,fd=9))
udp    UNCONN     0      0      [::]:111                [::]:*                   users:(("rpcbind",pid=339,fd=9))
udp    UNCONN     0      0      [::]:2049               [::]:*
tcp    LISTEN     0      64        *:2049                  *:*
tcp    LISTEN     0      128       *:111                   *:*                   users:(("rpcbind",pid=339,fd=8))
tcp    LISTEN     0      128       *:20048                 *:*                   users:(("rpc.mountd",pid=3551,fd=8))
tcp    LISTEN     0      64     [::]:2049               [::]:*
tcp    LISTEN     0      128    [::]:111                [::]:*                   users:(("rpcbind",pid=339,fd=11))
tcp    LISTEN     0      128    [::]:20048              [::]:*                   users:(("rpc.mountd",pid=3551,fd=10))
```
 Создаём и меняем права для директории, которая будет экспортирована:
 ```
$ mkdir -p /srv/share/upload
$ chown -R nfsnobody:nfsnobody /srv/share
$ chmod 0777 /srv/share/upload

Output:
[root@nfss ~]# mkdir -p /srv/share/upload

[root@nfss ~]# chown -R nfsnobody:nfsnobody /srv/share
[root@nfss ~]# ll /srv/share/upload/
total 0
[root@nfss ~]# ls -lah /srv/share/upload/
total 0
drwxrwxrwx. 2 nfsnobody nfsnobody  6 Nov 18 10:38 .
drwxr-xr-x. 3 nfsnobody nfsnobody 20 Nov 18 10:38 ..
[root@nfss ~]# chmod 0777 /srv/share/upload
 ```
 Создаём в файле /etc/export структуру, которая позволит экспортировать ранее созданную директорию:
 ```
$ cat << EOF > /etc/exports
/srv/share 192.168.50.11/32(rw,sync,root_squash)
EOF
Output:
[root@nfss ~]# cat << EOF > /etc/exports
> /srv/share 192.168.50.11/32(rw,sync,root_squash)
> EOF
 ```
Проверяем файл /etc/exports:
```
[root@nfss ~]# cat /etc/exports
/srv/share 192.168.50.11/32(rw,sync,root_squash)
```
Экспортируем ранее созданную директорию: 
```
$ exportfs -r
```
Проверяем экспортированную директорию следующей командой:
```
$ exportfs -s

Output:
[root@nfss ~]# exportfs -s
/srv/share  192.168.50.11/32(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
```

        #Настраиваем SERVER
        
Заходим на сервер CLIENT:
```
$ vagrant ssh nfsc
```
Получаем рут права:
```
$ sudo -i
```
Ставим NFS утилиты:
```
$ yum install nfs-utils -y
```

Включаем firewall и проверяем, что он работает:
```
$ systemctl enable firewalld --now
```
Output:
```
Created symlink from /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service to /usr/lib/systemd/system/firewalld.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/firewalld.service to /usr/lib/systemd/system/firewalld.service.

$ systemctl status firewalld

Output:

● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2022-11-18 10:50:10 UTC; 16s ago
     Docs: man:firewalld(1)
 Main PID: 3391 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─3391 /usr/bin/python2 -Es /usr/sbin/firewalld --nofork --nopid
```

Добавляем в /etc/fstab строку (это нужно сделать для автомонтирования /mnt)
```
$ echo "192.168.50.10:/srv/share/ /mnt nfs vers=3,proto=udp,noauto,x-systemd.automount 0 0" >> /etc/fstab

Output:

[root@nfsc ~]# cat /etc/fstab | grep 192.168.50.10
192.168.50.10:/srv/share/ /mnt nfs vers=3,proto=udp,noauto,x-systemd.automount 0 0
```
Перезапускаем сервисы:
```
$ systemctl daemon-reload
$ systemctl restart remote-fs.target

```
заходим в директорию `/mnt/` и проверяем успешность монтирования:
```
$ cd /mnt/
$ mount | grep mnt

Output:

[root@nfsc mnt]#  mount | grep mnt
systemd-1 on /mnt type autofs (rw,relatime,fd=46,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=26895)
192.168.50.10:/srv/share/ on /mnt type nfs (rw,relatime,vers=3,rsize=32768,wsize=32768,namlen=255,hard,proto=udp,timeo=11,retrans=3,sec=sys,mountaddr=192.168.50.10,mountvers=3,mountport=20048,mountproto=udp,local_lock=none,addr=192.168.50.10)
```
На это настройка Клиента завершается!

Заходим на сервер nfss:
```
$ vagrant ssh nfss
```
Переходим в каталог /srv/share/upload:
```
$ cd /srv/share/upload
```
Создаём тестовый файл:
```
$ ip addr | grep 192.168 >> ./check_file.txt

[root@nfss upload]# ls -lah
total 4.0K
drwxrwxrwx. 2 nfsnobody nfsnobody 28 Nov 18 11:08 .
drwxr-xr-x. 3 nfsnobody nfsnobody 20 Nov 18 10:38 ..
-rw-r--r--. 1 root      root      77 Nov 18 11:08 check_file.txt
[root@nfss upload]# cat check_file.txt
    inet 192.168.50.10/24 brd 192.168.50.255 scope global noprefixroute eth1
[root@nfss upload]#
```
Заходим на сервер CLIENT:
```
$ vagrant ssh nfsc
```
Переходим в каталог `/mnt/upload`:
```
$ cd /mnt/upload
```
Проверяем наличе файла созданного на сервере nfss:
```
[root@nfsc upload]# ls -lah
total 4.0K
drwxrwxrwx. 2 nfsnobody nfsnobody 28 Nov 18 11:08 .
drwxr-xr-x. 3 nfsnobody nfsnobody 20 Nov 18 10:38 ..
-rw-r--r--. 1 root      root      77 Nov 18 11:08 check_file.txt
[root@nfsc upload]# cat check_file.txt
    inet 192.168.50.10/24 brd 192.168.50.255 scope global noprefixroute eth1
[root@nfsc upload]#
```
Здесь же создаем файл:
```
$ ip addr | grep 192.168 >> ./check_file_from_client.txt

Output:
[root@nfsc upload]# ip addr | grep 192.168 >> ./check_file_from_client.txt
[root@nfsc upload]# ll
total 8
-rw-r--r--. 1 nfsnobody nfsnobody 77 Nov 18 11:12 check_file_from_client.txt
-rw-r--r--. 1 root      root      77 Nov 18 11:08 check_file.txt
[root@nfsc upload]# cat check_file_from_client.txt
    inet 192.168.50.11/24 brd 192.168.50.255 scope global noprefixroute eth1

```

        #Проверяем сервер nfss:
        
```
[root@nfss upload]# reboot
Connection to 127.0.0.1 closed by remote host.
Connection to 127.0.0.1 closed.
PS E:\centos\nfs> vagrant ssh nfss
Last login: Fri Nov 18 10:14:10 2022 from 10.0.2.2
[vagrant@nfss ~]$ ll /srv/share/upload/
total 8
-rw-r--r--. 1 nfsnobody nfsnobody 77 Nov 18 11:12 check_file_from_client.txt
-rw-r--r--. 1 root      root      77 Nov 18 11:08 check_file.txt
[vagrant@nfss ~]$ systemctl status nfs
● nfs-server.service - NFS server and services
   Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; enabled; vendor preset: disabled)
  Drop-In: /run/systemd/generator/nfs-server.service.d
           └─order-with-mounts.conf
   Active: active (exited) since Fri 2022-11-18 11:16:21 UTC; 43s ago
  Process: 828 ExecStartPost=/bin/sh -c if systemctl -q is-active gssproxy; then systemctl reload gssproxy ; fi (code=exited, status=0/SUCCESS)
  Process: 800 ExecStart=/usr/sbin/rpc.nfsd $RPCNFSDARGS (code=exited, status=0/SUCCESS)
  Process: 798 ExecStartPre=/usr/sbin/exportfs -r (code=exited, status=0/SUCCESS)
 Main PID: 800 (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/nfs-server.service
[vagrant@nfss ~]$ systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2022-11-18 11:16:18 UTC; 57s ago
     Docs: man:firewalld(1)
 Main PID: 402 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─402 /usr/bin/python2 -Es /usr/sbin/firewalld --nofork --nopid
[vagrant@nfss ~]$

```

        #Проверяем клиент nfsс:
```
[root@nfsc upload]# reboot
Connection to 127.0.0.1 closed by remote host.
Connection to 127.0.0.1 closed.
PS E:\centos\nfs> vagrant ssh nfsc
Last login: Fri Nov 18 10:16:33 2022 from 10.0.2.2
[vagrant@nfsc ~]$ showmount -a 192.168.50.10
All mount points on 192.168.50.10:
[vagrant@nfsc ~]$ cd /mnt/upload
[vagrant@nfsc upload]$ mount | grep mnt
systemd-1 on /mnt type autofs (rw,relatime,fd=27,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=10880)
192.168.50.10:/srv/share/ on /mnt type nfs (rw,relatime,vers=3,rsize=32768,wsize=32768,namlen=255,hard,proto=udp,timeo=11,retrans=3,sec=sys,mountaddr=192.168.50.10,mountvers=3,mountport=20048,mountproto=udp,local_lock=none,addr=192.168.50.10)
[vagrant@nfsc upload]$ mount | grep mnt >> ./final_check.txt
[vagrant@nfsc upload]$ cat final_check.txt
systemd-1 on /mnt type autofs (rw,relatime,fd=27,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=10880)
192.168.50.10:/srv/share/ on /mnt type nfs (rw,relatime,vers=3,rsize=32768,wsize=32768,namlen=255,hard,proto=udp,timeo=11,retrans=3,sec=sys,mountaddr=192.168.50.10,mountvers=3,mountport=20048,mountproto=udp,local_lock=none,addr=192.168.50.10)        
```
