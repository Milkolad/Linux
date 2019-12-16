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
++ -m создание домашней папки пользователя в каталоге home
++ -s имя регистрационной командной оболочки пользователя

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


