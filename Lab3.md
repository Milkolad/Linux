## Посмотрим какие дисковые устройства и партиции есть в системе

```bash
lsblk
blkid
fdisk -l 
parted -l
```
1. Какой размер дисков?
+ 931,5 гигабайт
2. Есть ли неразмеченное место на дисках?
+ нет
3. Какой размер партиций?
+ sda1 = 100M, sda2=128M, sda3=312.3G, sda4=619G 
4. Какая таблица партционирования используется?
+ GPT
5. Какой диск, партция или лвм том смонтированы в /
+ sda4

---

## Создадим сжатую файловую систему для чтения squashfs
```bash
git clone https://gitlab.com/erlong15/mai.git
mksquashfs mai/* mai.sqsh
sudo mkdir /mnt/mai
sudo mount mai.sqsh /mnt/mai -t squashfs -o loop
```

---

## Посмотрим информацию по файловым системам смонтированным в системе

```bash
df -h
df -i
mount
```
1. Какая файловая система примонтирована в /
+ /dev/sda4 
2. С какими опциями примонтирована файловая система в /
```bash
/dev/sda4 on / type ext4 (rw,relatime,errors=remount-ro)
```
3. Какой размер файловой системы приментированной в /mnt/mai
+  128K

---

## Попробуем создать файлик в каталоге /dev/shm

```bash
dd if=/dev/zero of=/dev/shm/mai bs=1M count=100
free -h
rm -f /dev/shm/mai
free -h
```
1. Что такое tmpfs
+ временное файловое хранилище
2. какая часть памяти изменялась?
+ общая, буфер


---

## Изучим процессы запущенные в системе

```bash
ps -eF
ps rx 
ps -e --forest
ps -efL
```

1. Какие процессы в системе порождают дочерние процессы через fork
+ kthreadd, NetworkManager, avahi-daemon, dockerd, gdm3, gdm-session-wor, gdm-wayland-ses, gnome-session-b, gnome-shell, ibus-daemon, gdm-session-wor, gdm-x-session, gnome-session-b, gnome-shell, ibus-daemon, systemd, at-spi-bus-laun, gvfsd, at-spi-bus-laun, evolution-calen, evolution-addre,  gnome-terminal-, bash, firefox
2. Какие процессы в системе являются мультитредовыми
+ kthreadd, gdm3, gnome-shell, ibus-daemon, gnome-session-b, gdm-x-session, gnome-session-b, ibus-daemon, systemd, firefox 



---

## Разберитесь что делает команда

```bash
ps axo rss | tail -n +2|paste -sd+ | bc
```

1. Что подсчитывается этой командой
+ сумма памяти используемой всеми процессами (размер памяти, выделенной каждому процессу операционной системой и в настоящее время находящейся в ОЗУ, начиная со второй по счету строки (только цифры), записать в одну строчку с "+" и выполнить вычисления)
2. Почему цифра такая странная
+ Потому что используется страничная память. Результат представлен в страницах. Если перевести: 5417052*4096/8/1024/1024/1024=2.6G памяти используется процессами

---

## Уставновим утилиту smem и проанализируем параметр PSS в ней
```bash
apt-get install smem -y
smem
```
```bash
user@node-14:~$ smem
  PID User     Command                         Swap      USS      PSS      RSS 
25475 user     /usr/bin/speech-dispatcher         0      436      460     3048 
 4924 user     /usr/bin/dbus-daemon --conf        0      580      681     4280 
 4970 user     /usr/libexec/xdg-permission        0      616      706     6036 
 5069 user     /usr/lib/gnome-settings-dae        0      680      737     6060 
 4773 user     /usr/lib/gdm3/gdm-x-session        0      728      769     6076 
 5055 user     /usr/lib/gvfs/gvfs-mtp-volu        0      804      853     6108 
 5064 user     /usr/lib/gnome-settings-dae        0      788      856     6296 
 5351 user     /usr/lib/gvfs/gvfsd-metadat        0      808      880     6308 
 5013 user     /usr/lib/dconf/dconf-servic        0      864      896     5276 
 5032 user     /usr/lib/gvfs/gvfs-goa-volu        0      824      925     6276 
 4926 user     /usr/lib/at-spi2-core/at-sp        0      820      929     7000 
25472 user     /usr/lib/speech-dispatcher-        0      776      951     5472 
25469 user     /usr/lib/speech-dispatcher-        0      796      960     5468 
 4979 user     /usr/lib/ibus/ibus-portal          0      976     1104     9128 
 5097 user     /usr/lib/gnome-settings-dae        0     1028     1108     8768 
 4919 user     /usr/lib/at-spi2-core/at-sp        0     1008     1115     8972 
 5116 user     /usr/lib/gnome-settings-dae        0     1048     1132     8908 
 4975 user     /usr/lib/ibus/ibus-dconf           0     1004     1134     9092 
 5051 user     /usr/lib/gvfs/gvfs-gphoto2-        0     1084     1160     6928 
 5234 user     /usr/lib/ibus/ibus-engine-s        0     1088     1190     9192 
 4903 user     /usr/lib/gvfs/gvfsd                0     1060     1213     7268 
 5103 user     /usr/lib/gnome-settings-dae        0     1148     1295     9620 
 5027 user     /usr/lib/gvfs/gvfs-afc-volu        0     1180     1401     8284 
 4908 user     /usr/lib/gvfs/gvfsd-fuse /r        0     1384     1446     8208 
 5089 user     /usr/lib/gnome-settings-dae        0     1408     1537    10696 
 4820 user     /usr/bin/dbus-daemon --sess        0     1528     1630     5284 
 5220 user     /usr/lib/gnome-disk-utility        0     1476     1648     6680 
 5061 user     /usr/lib/gnome-settings-dae        0     1452     1762    10572 
 5241 user     /usr/lib/gvfs/gvfsd-trash -        0     1724     1945    10628 
 5046 user     /usr/lib/gnome-online-accou        0     1980     2171    10844 
 5149 user     /usr/lib/gnome-settings-dae        0     1956     2207    13380 
 4755 user     /lib/systemd/systemd --user        0     1616     2359     8216 
25463 user     /usr/lib/speech-dispatcher-        0     2132     2372     7936 
 4967 user     ibus-daemon --xim --panel d        0     2268     2440    10796 
 5078 user     /usr/lib/gnome-settings-dae        0     1280     2448    11356 
26878 user     bash                               0     1916     2483     5468 
 8042 user     bash                               0     1980     2547     5532 
 5104 user     /usr/lib/gnome-settings-dae        0     2284     2587    15856 
 5019 user     /usr/lib/gvfs/gvfs-udisks2-        0     2332     2600    11784 
 5074 user     /usr/lib/gnome-settings-dae        0     2376     2754    12712 
 4823 user     /usr/lib/gnome-session/gnom        0     3024     3537    16876 
 5101 user     /usr/lib/gnome-settings-dae        0     4968     5391    24164 
 4977 user     /usr/lib/ibus/ibus-x11 --ki        0     4976     5462    24724 
 5113 user     /usr/lib/gnome-settings-dae        0     5260     5745    25508 
 4991 user     /usr/lib/gnome-shell/gnome-        0     4000     5767    22772 
 5088 user     /usr/lib/gnome-settings-dae        0     5380     5851    25296 
 5277 user     /usr/lib/evolution/evolutio        0     3868     5912    25084 
 5059 user     /usr/lib/gnome-settings-dae        0     5508     6033    26240 
 5558 user     update-notifier                    0     5524     6248    27832 
 5020 user     /usr/lib/evolution/evolutio        0     5128     6529    27468 
 5099 user     /usr/lib/gnome-settings-dae        0     5824     6668    27300 
 5086 user     /usr/lib/gnome-settings-dae        0     6324     6777    26036 
 5107 user     /usr/lib/gnome-settings-dae        0     5916     6789    28448 
 1055 user     /usr/bin/python /usr/bin/sm        0     6996     7027     9736 
 5286 user     /usr/lib/evolution/evolutio        0     5284     7931    28628 
 4952 user     /usr/bin/pulseaudio --start        0     8000     9912    17756 
 8033 user     /usr/lib/gnome-terminal/gno        0    13612    14938    42208 
 5036 user     /usr/lib/gnome-online-accou        0    16532    17276    33316 
15617 user     /usr/bin/nautilus --gapplic        0    16768    20513    52408 
 5207 user     nautilus-desktop                   0    15088    25717    63504 
30731 user     /usr/lib/firefox/firefox -c        0    19700    27801    84448 
 5865 user     /usr/lib/firefox/firefox -c        0    26196    37440   107652 
 5264 user     /usr/lib/evolution/evolutio        0    40288    43297    64212 
 5228 user     /usr/lib/evolution/evolutio        0    40604    43517    67736 
 4775 user     /usr/lib/xorg/Xorg vt2 -dis        0    43064    82151   135240 
 7566 user     /usr/lib/firefox/firefox -c        0    85308   101597   186472 
 5561 user     /usr/bin/gnome-software --g        0   101252   103735   134724 
 6628 user     /usr/lib/firefox/firefox -c        0   139652   159418   249248 
 4942 user     /usr/bin/gnome-shell               0   154868   170313   223816 
 6807 user     /usr/lib/firefox/firefox -c        0   156176   185650   285848 
 5595 user     /usr/lib/firefox/firefox -n        0   542040   583729   697540 
```

---

## Запустим приложеннный скрипт и понаблюдаем за процессами
```bash
python myfork.py
```
* в другом терминале  отследите порождение процессов
* отследите какие состояния вы видите у процессов
* почему появляются процессы со статусам Z
* какой PID у основного процесса
* убейте основной процесс ```kill -9 <pid>```
* какой PPID стал у первого чайлда
* насколько вы разобрались в скрипте и втом что он делает?

---

## Научимся корректно завершать зомби процессы
* запустим еще раз наш процесс
* убьем процесс первого чайлда
* проверим его состояние  и убедимся что он зомби
* остановим основной процесс
* расскоментируем строки в скрипте
```python
      pid, status = os.waitpid(pid, 0)
      print("wait returned, pid = %d, status = %d" % (pid, status))
```
* поторим все еще раз
* отследим корректное завершение чайлда

---
## Научимся убивать зомби процессы
* запускаем процессс еще раз
```bash
gdb
> attach <parent_PID>
> call waitpid(zombie_PID, 0, 0) wait
> detach
> quit
```

---
## Проблемы при отмонтировании директории
```bash
cd /mnt/mai
# в другом терминале
sudo umount /mnt/mai
fuser -v /mnt/dir
```
* Напишите какие процессы мешают размонтировать директорию
* посмотрите ```lsof -p <pid>``` этих процессов
* убейте мешаюший процесс и размонтируйте директорию

---

## Решаем загадку исчезновения места на диске
* создадим директорию ~/myfiles
* запустим файл test_write.py из ранее созданной директории
* проверим в другом терминале что в этой директории создался файл и он увеличивается в размере
* в первом терминали нажмем Ctrl+Z
* проверим статус файла
* выполним команды
```bash
jobs -l
fg
```
* eще раз остановим  процесс черерз CTRL+Z
* выполним команду ```bg```
* проверим размер файловой системы и каталога
```bash
df 
du -sh  ~/myfiles
```
* удалим файл
* повторно проверим размеры кталога и файловой системы
* Какую систуацию вы видите, как вы это объясните
* Подключитесь к процессу через ```strace -p <pid>``` и назовите дескриптор файла, куда пишет процесс
* проверим какие файлы открыты у нашего процесса через команду lsof -p <pid>
* убьем процесс
* еще раз прорим размер файловой системы и каталога
* Напишите свое объяснение, что произошло

---

## Утилиты наблюдения
* C помощью утилит мониторинга 
* проверьте текущий LA 
* запишите top 3 процессов загружающих CPU
* запишите top 3 процессов загружающих память 
* запустите утилиту atop как сервис через systemd
* запустите dd на генерацию файла размер в 3 гигабайта
* удалите сгенеренный файл
* через atop скажите какой  pid был у процесса
* Проанализируйте нагрузку на диск через утилиты  iotop и iostat
