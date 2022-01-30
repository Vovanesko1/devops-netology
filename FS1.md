# Домашнее задание к занятию "3.5. Файловые системы"

1. - Узнайте о sparse (разряженных) файлах.

https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D0%B7%D1%80%D0%B5%D0%B6%D1%91%D0%BD%D0%BD%D1%8B%D0%B9_%D1%84%D0%B0%D0%B9%D0%BB

Разрежённый файл (англ. sparse file) — файл, в котором последовательности нулевых байтов[1] заменены на информацию об этих последовательностях (список дыр).
Разрежённые файлы используются для хранения контейнеров, например:
образов дисков виртуальных машин;
резервных копий дисков и/или разделов, созданных спец. ПО.

2. - Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?
```sh
touch 1
ls -l
-rw-rw-r-- 1 vagrant vagrant       0 Jan 30 08:46  1
ln -P 1 2
chmod 777 2
ls -l
-rwxrwxrwx 2 vagrant vagrant       2 Jan 30 09:00  1
-rwxrwxrwx 2 vagrant vagrant       2 Jan 30 09:00  2
```
Разные права доступа на один объект, при использовании жесткой сылки, иметь нельзя. Т.к. файловая систем работает через индексы Inode, а он у них одинаковый. Команда 'chmod 777 2' по смыслу равна команде 'chmod 777 Inode: 1058980'
```
stat 2
Device: fd00h/64768d    Inode: 1058980     Links: 2
stat 1
Device: fd00h/64768d    Inode: 1058980     Links: 2
```
Кстати, на всех файлах параметр ' Birth: -' у меня в ВМ так же не заполнен как и на машине Александра, используемой в видео.

3. - Сделайте vagrant destroy на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:
```
Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-20.04"
  config.vm.provider :virtualbox do |vb|
    lvm_experiments_disk0_path = "/tmp/lvm_experiments_disk0.vmdk"
    lvm_experiments_disk1_path = "/tmp/lvm_experiments_disk1.vmdk"
    vb.customize ['createmedium', '--filename', lvm_experiments_disk0_path, '--size', 2560]
    vb.customize ['createmedium', '--filename', lvm_experiments_disk1_path, '--size', 2560]
    vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk0_path]
    vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk1_path]
  end
end
```
- Данная конфигурация создаст новую виртуальную машину с двумя дополнительными неразмеченными дисками по 2.5 Гб.
```
vagrant destroy
    default: Are you sure you want to destroy the 'default' VM? [y/N] y
==> default: Forcing shutdown of VM...
==> default: Destroying VM and associated drives...
```
Меняем файл 'G:\VirtualBox VMs\Vagrantfile' и 'vagrant up'

4. - Используя fdisk, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.

2 Гигабайта высчитано черз сектора. 5242879 секторов на диске по 512 байт. 2 Гигабайта = 2147483648 байт. Делим на размер сектора 512 и получаем 4194304 секторов.
```
sudo fdisk -l

sudo fdisk /dev/sdb
Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-5242879, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-5242879, default 5242879): 4194304

Created a new partition 1 of type 'Linux' and of size 2 GiB.

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2): 2
First sector (4194305-5242879, default 4196352):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-5242879, default 5242879):

Created a new partition 2 of type 'Linux' and of size 511 MiB.
w
```

5. - Используя sfdisk, перенесите данную таблицу разделов на второй диск.

```
sudo sfdisk -d /dev/sdb > sdb_tables
sudo sfdisk /dev/sdc < sdb_tables

sudo fdisk /dev/sdc
Command (m for help): p
Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x78a280d8

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdc1          2048 4194304 4192257    2G 83 Linux
/dev/sdc2       4196352 5242879 1046528  511M 83 Linux
```

6. - Соберите mdadm RAID1 на паре разделов 2 Гб.

видим наши диски: 
```
sudo fdisk -l
Device     Boot   Start     End Sectors  Size Id Type
/dev/sdb1          2048 4194304 4192257    2G 83 Linux
/dev/sdb2       4196352 5242879 1046528  511M 83 Linux

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdc1          2048 4194304 4192257    2G 83 Linux
/dev/sdc2       4196352 5242879 1046528  511M 83 Linux
```
/dev/md1 — устройство RAID, которое появится после сборки; 
-l 1 — уровень RAID; 
-n 2 — количество дисков, из которых собирается массив; 
/dev/sd{b1,c1} — сборка выполняется из дисков sdb и sdc.
```
sudo mdadm --create --verbose /dev/md1 -l 1 -n 2 /dev/sd{b1,c1}
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
mdadm: size set to 2094080K
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.

sudo fdisk -l
Disk /dev/md1: 1.102 GiB, 2144337920 bytes, 4188160 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

7. - Соберите mdadm RAID0 на второй паре маленьких разделов.
```
sudo mdadm --create --verbose /dev/md0 -l 0 -n 2 /dev/sd{b2,c2}
sudo fdisk -l
Disk /dev/md0: 1018 MiB, 1067450368 bytes, 2084864 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 524288 bytes / 1048576 bytes
```

8. - Создайте 2 независимых PV на получившихся md-устройствах.

Структура LVM состоит из трех слоев:
Физический том (один или несколько), Physical Volume (PV)
Группа физических томов, Volume Group (VG)
Логический том, который и будет доступен программам, Logical Volume (LV)

смотрим существующие физические разделы
```
sudo pvs
  PV         VG        Fmt  Attr PSize   PFree
  /dev/sda3  ubuntu-vg lvm2 a--  <63.00g <31.50g
```
Создаём 2 независимых PV на получившихся md-устройствах.
```
 sudo pvcreate /dev/md0 /dev/md1
  Physical volume "/dev/md0" successfully created.
  Physical volume "/dev/md1" successfully created.
   sudo pvs
  PV         VG        Fmt  Attr PSize    PFree
  /dev/md0             lvm2 ---  1018.00m 1018.00m
  /dev/md1             lvm2 ---    <2.00g   <2.00g
  /dev/sda3  ubuntu-vg lvm2 a--   <63.00g  <31.50g
```

9. - Создайте общую volume-group на этих двух PV.

Группа томов - это не что иное, как пул памяти, который будет распределен между логическими томами и может состоять из нескольких физических разделов. После того как физические разделы инициализированы, можно создать из них группу томов (Volume Group, VG)
```
sudo vgcreate vol_grp1 /dev/md0  /dev/md1
Volume group "vol_grp1" successfully created

sudo vgdisplay
  --- Volume group ---
  VG Name               vol_grp1
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               <2.99 GiB
  PE Size               4.00 MiB
  Total PE              765
  Alloc PE / Size       0 / 0
  Free  PE / Size       765 / <2.99 GiB
  VG UUID               H0I6bT-sRNV-I2rp-bwie-OvUt-AJ1J-i7mCBG
```
10. - Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.
```
 sudo lvcreate -L 100M vol_grp1 /dev/md0
  Logical volume "lvol0" created.
```
В ответе не уверен. Сработала опция '/dev/md0' или пропущена?
Однако если указать не верный путь к файлу устройства идёт ошибка
sudo lvcreate -L 100M vol_grp1 /dev/md0000
  Physical Volume "/dev/md0000" not found in Volume Group "vol_grp1".

В выводе информации нет указания на каком рейде живёт наш объем "lvol0":
```
sudo lvdisplay -a
 --- Logical volume ---
  LV Path                /dev/vol_grp1/lvol0
  LV Name                lvol0
  VG Name                vol_grp1
  LV UUID                3XNlpy-gMXQ-hYZW-gI24-UKdU-75kB-xS5Db1
  LV Write Access        read/write
  LV Creation host, time vagrant, 2022-01-30 12:51:47 +0000
  LV Status              available
  # open                 0
  LV Size                100.00 MiB
  Current LE             25
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  currently set to     4096
  Block device           253:1
```

11. - Создайте mkfs.ext4 ФС на получившемся LV.
```
sudo mkfs.ext4 /dev/vol_grp1/lvol0
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done
Writing inode tables: done
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done
```

12. - Смонтируйте этот раздел в любую директорию, например, /tmp/new.

Монтируем без сохранения точки монтирования при перезагрузке:
```
mkdir /tmp/new
sudo mount /dev/vol_grp1/lvol0 /tmp/new
```

13. - Поместите туда тестовый файл, например wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz.
```
sudo wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz
/tmp/new$ ls
lost+found  test.gz
```

14. - Прикрепите вывод lsblk.
```
 lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                       7:0    0 55.4M  1 loop  /snap/core18/2128
loop1                       7:1    0 70.3M  1 loop  /snap/lxd/21029
loop2                       7:2    0 32.3M  1 loop  /snap/snapd/12704
loop3                       7:3    0 55.5M  1 loop  /snap/core18/2284
loop4                       7:4    0 43.4M  1 loop  /snap/snapd/14549
loop5                       7:5    0 61.9M  1 loop  /snap/core20/1328
loop6                       7:6    0 67.2M  1 loop  /snap/lxd/21835
sda                         8:0    0   64G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    1G  0 part  /boot
└─sda3                      8:3    0   63G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.5G  0 lvm   /
sdb                         8:16   0  2.5G  0 disk
├─sdb1                      8:17   0    2G  0 part
│ └─md1                     9:1    0    2G  0 raid1
└─sdb2                      8:18   0  511M  0 part
  └─md0                     9:0    0 1018M  0 raid0
    └─vol_grp1-lvol0      253:1    0  100M  0 lvm   /tmp/new
sdc                         8:32   0  2.5G  0 disk
├─sdc1                      8:33   0    2G  0 part
│ └─md1                     9:1    0    2G  0 raid1
└─sdc2                      8:34   0  511M  0 part
  └─md0                     9:0    0 1018M  0 raid0
    └─vol_grp1-lvol0      253:1    0  100M  0 lvm   /tmp/new
```

15. - Протестируйте целостность файла:
```
root@vagrant:~# gzip -t /tmp/new/test.gz
root@vagrant:~# echo $?
0
```

16. - Используя pvmove, переместите содержимое PV с RAID0 на RAID1.

pvmove позволяет переместить данные с одного физического тома на другой.
```
sudo pvmove /dev/md0 /dev/md1
  /dev/md0: Moved: 24.00%
  /dev/md0: Moved: 100.00%
```
17. - Сделайте --fail на устройство в вашем RAID1 md.

Диск может быть помечен как неисправный либо из-за сбоя, либо если вы хотите вручную пометить диск как неисправный, используйте флаг -f. Указываем рейд и диск
```
sudo mdadm -f /dev/md1 /dev/sdc1
```

18. - Подтвердите выводом dmesg, что RAID1 работает в деградированном состоянии.
```
dmesg
[15370.769392] md/raid1:md1: Disk failure on sdc1, disabling device.
               md/raid1:md1: Operation continuing on 1 devices.
```

19. - Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:
```
vagrant@vagrant:/tmp/new$ gzip -t /tmp/new/test.gz
vagrant@vagrant:/tmp/new$ echo $?
0
```
20. - Погасите тестовый хост, vagrant destroy.

Пусть поживёт.
```
vagrant halt
```




















