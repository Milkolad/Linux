# Отчет по лабораторной работе 2

## Создаем пользователей
```bash
[root@centos7-vm vagrant]\# useradd -m -s /bin/bash user1
[root@centos7-vm vagrant]\# useradd -m -s /bin/bash user2
```

* Какие UID создались у пользователей?
```bash
uid=1002(user1)
uid=1003(user2)
```
* Что означают опции -m и -s
1. -m создание домашней папки пользователя в каталоге home
2. -s имя регистрационной командной оболочки пользователя

---
## Создаем группу и добавляем туда пользователей
```bash
man groupadd
man gpasswd
groupadd admins
gpasswd -a user1 admins
gpasswd -a user2 admins
id user1
id user2
```
результат:
```bash
uid=1002(user1) gid=1002(user1) группы=1002(user1),1004(admins)
uid=1003(user2) gid=1003(user2) группы=1003(user2),1004(admins)
```
+ (*) Через usermod сделайте группу admins основной для  user1
```bash
uid=1002(user1) gid=1004(admins) группы=1004(admins)
uid=1003(user2) gid=1004(admins) группы=1004(admins)
```
---
##Создать каталог от рута и дать права группе admins туда писать
```bash
  mkdir /opt/upload
chmod 770 /opt/upload
chgrp admins /opt/upload
```
+ Что означают права 770 ?
1. Владельцу все права, группе все права, остальным никаких прав
* создать по файлу от пользователей user1 и user2 в каталоге /opt/uploads
* проверьте с какой группой создались файлы от каждого пользователя. Как думаете - почему?
1. admins, так как это основная группа пользователей, создавших файлы. (users - потому что мы временно изменили основную группу)
* (*) попробуйте сменить текущую группу пользователя  ```newgrp admins``` у пользователя user2 и создайте еще файл
* приложить ```ls -l /opt/upload```  в  README.md
```bash
-rw-r--r-- 1 user2 users  0 ноя 20 20:01 file2user2
-rw-r--r-- 1 user1 admins 0 ноя 20 19:28 fileuser1
-rw-r--r-- 1 user2 admins 0 ноя 20 19:29 fileuser2
```
---

## Создать пользователя user3 и дать ему права писать в /opt/uploads

* Создайте пользователя user3
* Попробуйте записать из под него файл в /opt/uploads. Должны получить ошибку
* Считайте acl с каталога. Добавьте черерз  setfacl права на запись в каталог.
```bash
man getfacl
man setfacl
getfacl /opt/upload
setfacl -m u:user3:rwx /opt/upload
su - user3
touch /opt/upload/user3_file
ls -l /opt/upload/user3_file
```
* приложить ```ls -l /opt/upload```  в  README.md
```bash
-rw-r--r-- 1 user2 users  0 ноя 20 20:01 file2user2
-rw-r--r-- 1 user1 admins 0 ноя 20 19:28 fileuser1
-rw-r--r-- 1 user2 admins 0 ноя 20 19:29 fileuser2
-rw-rw-r-- 1 user3 user3  0 ноя 20 20:20 fileuser3
```
* приложить финишный acl  директории в README.md
```bash
# file: opt/upload
# owner: root
# group: admins
user::rwx
user:user3:rwx
group::rwx
mask::rwx
other::---
```
---

## Установить GUID флаг на директорию /opt/uploads

_текущие версии Linux игнорируют установку SUID на диреторию, также игнорируется выставление SUID на shell скрипт_

```bash
chmod g+s /opt/upload
su - user3
touch /opt/upload/user3_file2
ls -l /opt/upload/user3_file2
```
* Приложить ```ls -l /opt/upload```  в  README.md
```bash
-rw-r--r-- 1 user2 users  0 ноя 20 20:01 file2user2
-rw-rw-r-- 1 user3 admins 0 ноя 20 20:22 file2user3
-rw-r--r-- 1 user1 admins 0 ноя 20 19:28 fileuser1
-rw-r--r-- 1 user2 admins 0 ноя 20 19:29 fileuser2
-rw-rw-r-- 1 user3 user3  0 ноя 20 20:20 fileuser3
```
* Объяснить почему изменилась группа при создании
1. потому что после выполнения chmod g+s все вновь созданные файлы в этом каталоге наследуют идентификатор группы в каталоге, а не идентификатор группы пользователя, который создал файл
---
## Установить  SUID  флаг на выполняемый файл

_текущие версии Linux игнорируют выставление SUID на shell скрипт (проеврка на shebang)_

* Установим suid бит на просмотрщик cat 
* В начале  попробуйте прочитать cat /etc/shadow  из под пользователя user3
```bash
cat: /etc/shadow: Отказано в доступе
```
* Установить suid /bin/cat и прочитайте снова из под user3
```bash
	root:$6$TLbz4FUH$gddh8Y1Fq4vgP12bPvoqWTxc1NExOil9y0dmClWo9CU7SwUCuFZ9RhaWdQMWeNSSPBNROf5Gj52oo2Gb9xb5p/:18178:0:99999:7:::
	daemon:*:18113:0:99999:7:::
	bin:*:18113:0:99999:7:::
	sys:*:18113:0:99999:7:::
	sync:*:18113:0:99999:7:::
	games:*:18113:0:99999:7:::
	man:*:18113:0:99999:7:::
	lp:*:18113:0:99999:7:::
	mail:*:18113:0:99999:7:::
	news:*:18113:0:99999:7:::
	uucp:*:18113:0:99999:7:::
	proxy:*:18113:0:99999:7:::
	www-data:*:18113:0:99999:7:::
	backup:*:18113:0:99999:7:::
	list:*:18113:0:99999:7:::
	irc:*:18113:0:99999:7:::
	gnats:*:18113:0:99999:7:::
	nobody:*:18113:0:99999:7:::
	systemd-network:*:18113:0:99999:7:::
	systemd-resolve:*:18113:0:99999:7:::
	syslog:*:18113:0:99999:7:::
	messagebus:*:18113:0:99999:7:::
	_apt:*:18113:0:99999:7:::
	uuidd:*:18113:0:99999:7:::
	avahi-autoipd:*:18113:0:99999:7:::
	usbmux:*:18113:0:99999:7:::
	dnsmasq:*:18113:0:99999:7:::
	rtkit:*:18113:0:99999:7:::
	cups-pk-helper:*:18113:0:99999:7:::
	speech-dispatcher:!:18113:0:99999:7:::
	whoopsie:*:18113:0:99999:7:::
	kernoops:*:18113:0:99999:7:::
	saned:*:18113:0:99999:7:::
	pulse:*:18113:0:99999:7:::
	avahi:*:18113:0:99999:7:::
	colord:*:18113:0:99999:7:::
	hplip:*:18113:0:99999:7:::
	geoclue:*:18113:0:99999:7:::
	gnome-initial-setup:*:18113:0:99999:7:::
	gdm:*:18113:0:99999:7:::
	user:			$6$XyHyFGcc$VvauulzXfqx25Z9GFP.wMriMT8LKG8p.RlzP7t6epouDgt6SmI2Ku.N9IjQCpyiqWePpGLWd67fgxgj4/onYb0:18151:0:99999:7:::
	statd:*:18178:0:99999:7:::
	user_polinanov:$6$6wHcRPPA$xR5TURzj8liYQi.rpJFA2c7q3aTFzDLyTxMz0SKFybvsQaLbgTcICTtXbBDrxvjGSE7G39okydl5SR8igICc./:18192:0:99999:7:::
	user1:$6$kcaorKjE$wDguv5SqbNqMOmIpIibvudcqYVMHGlL6DalsdDe/XXll/xBQfRjNKcF066L52FanOp03l3fmK4b8xEOVmyleb.:18220:0:99999:7:::
	user2:$6$UIxFC64B$NiK7Wz5eA6pTUBsmpcRE5ZaesE6Jjt/iNW6As58P9QtnmZ663PE4LAtKLKM5Y4tqRz3rnrXVKmThEbnejVrrz0:18220:0:99999:7:::
	user3:!:18220:0:99999:7:::
```
* Объясните почему
1. потому что теперь любой пользователь может выполнять cat из под рута (chmod u+s /bin/cat)

---
 
 
##  Сменить владельца  /opt/uploads  на user3 и добавить sticky bit
```bash
chown user3 /opt/upload
chmod +t /opt/upload
su - user1
touch /opt/upload/user1_file_test
ls -l /opt/upload/user1_file_test
su - user3
rm -f  /opt/upload/user1_file_test
```
* Объясните почему user3 смог удалить файл, который ему не принадлежит
1. user3 смог удалить файл user1 потому что он теперь владелец этого каталога (chown user3)
```bash	
# file: opt/upload
# owner: user3
# group: admins
# flags: -st
user::rwx
user:user3:rwx
group::rwx
mask::rwx
other::---
```
* Создайте теперь файл от user1 и удалите его пользователем user1
1. user1 может удалить файл который он же создал, потому что он его владелец
* Объясните результат
1. потому что user3 не входит в группу, которой разрешено выполнять sudo

---

## Записи в sudoers
* попробуйте из под user3 выполнить ```sudo ls -l /root```
```bash
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for user3:
user3 is not in the sudoers file.  This incident will be reported.
```
* для редактирования sudoers используйте  visudo
* почему у вас не получилось?
1. user3 не может использовать sudo и не имеет прав для просмотра каталога root
```bash
vi /etc/sudoers.d/user3
user3	ALL=NOPASSWD:/bin/ls
```

* добавьте запись в /etc/sudoers.d/admins разрешающий группе admins любые команды с вводом пароля
```bash
total 4
drwxr-xr-x 3 root root 4096 ноя 17 14:40 snap
```
