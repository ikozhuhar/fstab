# fdisk - способ управления разделами дисков в Linux

## <a name='toc'>Содержание</a>

1. [Как ядро Linux работает с жесткими дисками?](#1)
2. [Что такое fdisk?](#2)
3. [Как вывести список доступных дисков с помощью команды fdisk?](#3)
4. [Как вывести список определенных разделов диска с помощью команды fdisk?](#4)
5. [Как вывести список доступных действий для команды fdisk?](#5)
6. [Как вывести список типов разделов с помощью команды fdisk?](#6)
7. [Как создать раздел на диске с помощью команды fdisk?](#7)
8. [Как создать расширенный раздел диска с помощью команды fdisk?](#8)
9. [Как просмотреть нераспределенное место на диске с помощью команды fdisk?](#9)
10. [Как создать логический раздел с помощью команды fdisk?](#10)
11. [Как удалить раздел с помощью команды fdisk?](#11)
12. [Как отформатировать раздел или создать файловую систему на разделе?](#12)
13. [Как смонтировать раздел в Linux?](#13)
14. [Дополнительные источники](#14)


Жесткие диски можно разделить на один или несколько логических дисков, называемых разделами. Это разделение описано в таблице разделов (MBR или GPT) в секторе 0 диска.

Linux нужен как минимум один раздел, а именно для его корневой файловой системы, и мы не можем установить Linux на диск без разделов.

После создания раздел должен быть отформатирован в соответствующей файловой системе, прежде чем в него можно будет записывать файлы. Нам понадобится какая-нибудь утилита для выполнения этого действия в Linux.

Для этого в Linux доступно множество утилит. Мы писали о Parted Command в прошлом, и сегодня мы будем обсуждать fdisk. Команда fdisk является одним из лучших инструментов для управления разделами диска в Linux. Она поддерживает диски максимум 2 ТБ, и все предпочитают использовать fdisk.

Эта утилита используется большим количеством администраторов Linux, потому что мы практически не используем более 2 ТБ сегодня из-за LVM и SAN. Она используется в большей части инфраструктуры Linux по всему миру. Тем не менее, если вы хотите создать большие разделы, например, более 2 ТБ, вам нужно использовать команду Parted или команду cfdisk.


## [[⬆]](#toc) <a name='1'>Как ядро Linux работает с жесткими дисками?</a>

Как человек, мы можем легко понять разные вещи, но компьютер нуждается в правильном преобразовании имен, чтобы понять все и вся.

В Linux устройства расположены в разделе /dev, и ядро понимает жесткий диск в следующем формате.

/dev/hdX[a-z]: IDE-диск с именем hdX в Linux
/dev/sdX[a-z]: SCSI-диск с именем sdX в Linux
/dev/xdX[a-z]: XT-диск с именем xdX в Linux
/dev/vdX[a-z]: виртуальный жесткий диск с именем vdX в Linux
/dev/fdN: гибкий диск с именем fdN в Linux
/dev/scdN or /dev/srN: CD-ROM с именем /dev/scdN или /dev/srN в Linux

## [[⬆]](#toc) <a name='2'>Что такое fdisk?

fdisk обозначает "fixed disk" или "format disk". Это утилита командной строки, которая позволяет пользователям выполнять различные действия с дисками. Она позволяет нам просматривать, создавать, изменять размеры, удалять, перемещать и копировать разделы.

Она понимает таблицы разделов MBR, Sun, SGI и BSD, не понимает таблицу разделов GUID (GPT) и не предназначена для больших разделов.

fdisk позволяет нам создать максимум четыре основных раздела на диск. Один из них может быть расширенным разделом, и он может содержать несколько логических разделов.

1-4 зарезервировано для четырех основных разделов, а логические разделы начинаются с 5.


## [[⬆]](#toc) <a name='3'>Как вывести список доступных дисков с помощью команды fdisk?</a>

Сначала мы должны узнать, какие диски были добавлены в систему, прежде чем выполнять какие-либо действия. Для просмотра списка всех доступных дисков в вашей системе выполните приведенную ниже команду. Она выведет возможные сведения о дисках, такие как имя диска, количество разделов на нем, размер диска, тип метки диска, идентификатор диска, идентификатор раздела и тип раздела.

```
$ sudo fdisk -l
Disk /dev/sda: 30 GiB, 32212254720 bytes, 62914560 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xeab59449

Device     Boot    Start      End  Sectors Size Id Type
/dev/sda  *    20973568 62914559 41940992  20G 83 Linux

Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/sdc: 10 GiB, 10737418240 bytes, 20971520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/sdd: 10 GiB, 10737418240 bytes, 20971520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/sde: 10 GiB, 10737418240 bytes, 20971520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

## [[⬆]](#toc) <a name='4'>Как вывести список определенных разделов диска с помощью команды fdisk?</a>

Если вы хотите увидеть определенный диск и его разделы, используйте следующий формат команды:

```
$ sudo fdisk -l /dev/sda
Disk /dev/sda: 30 GiB, 32212254720 bytes, 62914560 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xeab59449

Device     Boot    Start      End  Sectors Size Id Type
/dev/sda  *    20973568 62914559 41940992  20G 83 Linux
```

## [[⬆]](#toc) <a name='5'>Как вывести список доступных действий для команды fdisk?</a>

Если вы нажмете m в команде fdisk, вы увидите доступные действия.

```
$ sudo fdisk /dev/sdc

Welcome to fdisk (util-linux 2.30.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xe944b373.

Command (m for help): m

Help:

  DOS (MBR)
   a   toggle a bootable flag
   b   edit nested BSD disklabel
   c   toggle the dos compatibility flag

  Generic
   d   delete a partition
   F   list free unpartitioned space
   l   list known partition types
   n   add a new partition
   p   print the partition table
   t   change a partition type
   v   verify the partition table
   i   print information about a partition

  Misc
   m   print this menu
   u   change display/entry units
   x   extra functionality (experts only)

  Script
   I   load disk layout from sfdisk script file
   O   dump disk layout to sfdisk script file

  Save & Exit
   w   write table to disk and exit
   q   quit without saving changes

  Create a new label
   g   create a new empty GPT partition table
   G   create a new empty SGI (IRIX) partition table
   o   create a new empty DOS partition table
   s   create a new empty Sun partition table
```

## [[⬆]](#toc) <a name='6'>Как вывести список типов разделов с помощью команды fdisk?</a>

Если вы нажмете l в команде fdisk, она покажет вам доступные типы разделов.
```
$ sudo fdisk /dev/sdc

Welcome to fdisk (util-linux 2.30.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x9ffd00db.

Command (m for help): l

 0  Empty           24  NEC DOS         81  Minix / old Lin bf  Solaris        
 1  FAT12           27  Hidden NTFS Win 82  Linux swap / So c1  DRDOS/sec (FAT-
 2  XENIX root      39  Plan 9          83  Linux           c4  DRDOS/sec (FAT-
 3  XENIX usr       3c  PartitionMagic  84  OS/2 hidden or  c6  DRDOS/sec (FAT-
 4  FAT16 <32M      40  Venix 80286     85  Linux extended  c7  Syrinx         
 5  Extended        41  PPC PReP Boot   86  NTFS volume set da  Non-FS data    
 6  FAT16           42  SFS             87  NTFS volume set db  CP/M / CTOS / .
 7  HPFS/NTFS/exFAT 4d  QNX4.x          88  Linux plaintext de  Dell Utility   
 8  AIX             4e  QNX4.x 2nd part 8e  Linux LVM       df  BootIt         
 9  AIX bootable    4f  QNX4.x 3rd part 93  Amoeba          e1  DOS access     
 a  OS/2 Boot Manag 50  OnTrack DM      94  Amoeba BBT      e3  DOS R/O        
 b  W95 FAT32       51  OnTrack DM6 Aux 9f  BSD/OS          e4  SpeedStor      
 c  W95 FAT32 (LBA) 52  CP/M            a0  IBM Thinkpad hi ea  Rufus alignment
 e  W95 FAT16 (LBA) 53  OnTrack DM6 Aux a5  FreeBSD         eb  BeOS fs        
 f  W95 Ext'd (LBA) 54  OnTrackDM6      a6  OpenBSD         ee  GPT            
10  OPUS            55  EZ-Drive        a7  NeXTSTEP        ef  EFI (FAT-12/16/
11  Hidden FAT12    56  Golden Bow      a8  Darwin UFS      f0  Linux/PA-RISC b
12  Compaq diagnost 5c  Priam Edisk     a9  NetBSD          f1  SpeedStor      
14  Hidden FAT16 <3 61  SpeedStor       ab  Darwin boot     f4  SpeedStor      
16  Hidden FAT16    63  GNU HURD or Sys af  HFS / HFS+      f2  DOS secondary  
17  Hidden HPFS/NTF 64  Novell Netware  b7  BSDI fs         fb  VMware VMFS    
18  AST SmartSleep  65  Novell Netware  b8  BSDI swap       fc  VMware VMKCORE 
1b  Hidden W95 FAT3 70  DiskSecure Mult bb  Boot Wizard hid fd  Linux raid auto
1c  Hidden W95 FAT3 75  PC/IX           bc  Acronis FAT32 L fe  LANstep        
1e  Hidden W95 FAT1 80  Old Minix       be  Solaris boot    ff  BBT
```

## [[⬆]](#toc) <a name='7'>Как создать раздел на диске с помощью команды fdisk?</a>

Если вы хотите создать новый раздел, выполните следующие действия. В моем случае я собираюсь создать 4 раздела (3 основных и 1 расширенный) на диске /dev/sdc.

Поскольку он принимает значение из таблицы разделов, нажмите Enter для первого сектора. Введите размер, который вы хотите установить для раздела (мы можем добавить размер раздела, используя КБ, МБ, ГБ и ТБ) для последнего сектора.

Например, если вы хотите добавить раздел размером 1 ГБ, последним значением сектора должно быть + 1G. Как только вы создали 3 раздела, fdisk автоматически изменит тип раздела на расширенный по умолчанию. Если вы все еще хотите создать четвертый первичный раздел, тогда нажмите p вместо значения по умолчанию e.

```
$ sudo fdisk /dev/sdc

Welcome to fdisk (util-linux 2.30.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): Enter

Using default response p.
Partition number (1-4, default 1): Enter
First sector (2048-20971519, default 2048): Enter
Last sector, +sectors or +size{K,M,G,T,P} (2048-20971519, default 20971519): +1G

Created a new partition 1 of type 'Linux' and of size 1 GiB.

Command (m for help): p
Disk /dev/sdc: 10 GiB, 10737418240 bytes, 20971520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x8cc8f9e5

Device     Boot Start     End Sectors Size Id Type
/dev/sdc1        2048 2099199 2097152   1G 83 Linux

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

## [[⬆]](#toc) <a name='8'>Как создать расширенный раздел диска с помощью команды fdisk?</a>

Обратите внимание, что вы должны использовать все оставшееся пространство при создании расширенного раздела, потому что вы затем можете создать в нем несколько логических разделов.

```
$ sudo fdisk /dev/sdc

Welcome to fdisk (util-linux 2.30.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): n
Partition type
   p   primary (3 primary, 0 extended, 1 free)
   e   extended (container for logical partitions)
Select (default e): Enter

Using default response e.
Selected partition 4
First sector (6293504-20971519, default 6293504): Enter
Last sector, +sectors or +size{K,M,G,T,P} (6293504-20971519, default 20971519): Enter

Created a new partition 4 of type 'Extended' and of size 7 GiB.

Command (m for help): p
Disk /dev/sdc: 10 GiB, 10737418240 bytes, 20971520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x8cc8f9e5

Device     Boot   Start      End  Sectors Size Id Type
/dev/sdc1          2048  2099199  2097152   1G 83 Linux
/dev/sdc2       2099200  4196351  2097152   1G 83 Linux
/dev/sdc3       4196352  6293503  2097152   1G 83 Linux
/dev/sdc4       6293504 20971519 14678016   7G  5 Extended

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

## [[⬆]](#toc) <a name='9'>Как просмотреть нераспределенное место на диске с помощью команды fdisk?</a>

Как описано в предыдущем разделе, мы полностью создали 4 раздела (3 основных и 1 расширенный). Дисковое пространство расширенного раздела будет отображаться как неразмеченное, пока вы не создадите в нем логические разделы.

Используйте приведенную ниже команду для просмотра неразмеченного пространства для диска. В соответствии с приведенным ниже выводом у нас есть 7ГБ неразмеченного диска.

```
$ sudo fdisk /dev/sdc

Welcome to fdisk (util-linux 2.30.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): F
Unpartitioned space /dev/sdc: 7 GiB, 7515144192 bytes, 14678016 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes

  Start      End  Sectors Size
6293504 20971519 14678016   7G

Command (m for help): q
```

## [[⬆]](#toc) <a name='10'>Как создать логический раздел с помощью команды fdisk?</a>

Выполните ту же процедуру, описанную выше, чтобы создать логический раздел после создания расширенного раздела. Здесь я создал логический раздел размером 1 ГБ с именем /dev/sdc5, вы можете убедиться в этом, проверив значение таблицы разделов.
```
$ sudo fdisk /dev/sdc

Welcome to fdisk (util-linux 2.30.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): n
All primary partitions are in use.
Adding logical partition 5
First sector (6295552-20971519, default 6295552): Enter
Last sector, +sectors or +size{K,M,G,T,P} (6295552-20971519, default 20971519): +1G

Created a new partition 5 of type 'Linux' and of size 1 GiB.

Command (m for help): p
Disk /dev/sdc: 10 GiB, 10737418240 bytes, 20971520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x8cc8f9e5

Device     Boot   Start      End  Sectors Size Id Type
/dev/sdc1          2048  2099199  2097152   1G 83 Linux
/dev/sdc2       2099200  4196351  2097152   1G 83 Linux
/dev/sdc3       4196352  6293503  2097152   1G 83 Linux
/dev/sdc4       6293504 20971519 14678016   7G  5 Extended
/dev/sdc5       6295552  8392703  2097152   1G 83 Linux

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

## [[⬆]](#toc) <a name='11'>Как удалить раздел с помощью команды fdisk?</a>

Если раздел больше не используется в системе, мы можем удалить его, используя следующие шаги. Убедитесь, что вы будете вводить правильный номер раздела, чтобы удалить его. В данном случае я собираюсь удалить раздел /dev/sdc2.
```
$ sudo fdisk /dev/sdc

Welcome to fdisk (util-linux 2.30.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): d
Partition number (1-5, default 5): 2

Partition 2 has been deleted.

Command (m for help): p
Disk /dev/sdc: 10 GiB, 10737418240 bytes, 20971520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x8cc8f9e5

Device     Boot   Start      End  Sectors Size Id Type
/dev/sdc1          2048  2099199  2097152   1G 83 Linux
/dev/sdc3       4196352  6293503  2097152   1G 83 Linux
/dev/sdc4       6293504 20971519 14678016   7G  5 Extended
/dev/sdc5       6295552  8392703  2097152   1G 83 Linux

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

## [[⬆]](#toc) <a name='12'>Как отформатировать раздел или создать файловую систему на разделе?</a>

Файловая система контролирует, как хранятся и извлекаются данные через таблицы inode. Без файловой системы система не может найти, где хранится информация на разделе. Файловая система может быть создана тремя способами. Здесь я собираюсь создать файловую систему в разделе /dev/sdc1.

```
$ sudo mkfs.ext4 /dev/sdc1
или

$ sudo mkfs -t ext4 /dev/sdc1
или

$ sudo mke2fs /dev/sdc1
mke2fs 1.43.5 (04-Aug-2017)
Creating filesystem with 262144 4k blocks and 65536 inodes
Filesystem UUID: c0a99b51-2b61-4f6a-b960-eb60915faab0
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done
Когда вы создаете файловую систему на этом разделе, это даст вам следующие важные вещи.
```

Когда вы создаете файловую систему на этом разделе, это даст вам следующие важные вещи.

**UUID файловой системы:** UUID означает универсальный уникальный идентификатор, UUID используются для идентификации блочных устройств в Linux. Это 128-битные числа, представленные 32 шестнадцатеричными цифрами.
**Superblock:** в суперблоке хранятся метаданные файловой системы. Если суперблок файловой системы поврежден, файловая система не может быть смонтирована и, следовательно, файлы не доступны.
**Inode:** это структура данных в файловой системе Unix-подобной операционной системы, в которой хранится вся информация о файле, кроме его имени и его фактических данных.
**Journal:** журналируемая файловая система - это файловая система, которая поддерживает специальный файл, называемый журналом, который используется для исправления любых несоответствий, возникающих в результате неправильного выключения компьютера.

## [[⬆]](#toc) <a name='13'>Как смонтировать раздел в Linux?</a>

После того, как вы создали раздел и файловую систему, нам нужно смонтировать раздел для использования. Для этого нам нужно создать точку монтирования для монтирования раздела. Используйте команду mkdir для создания точки монтирования.

```
$ sudo mkdir -p /mnt/2g-new
```
Для временного монтирования используйте приведенную ниже команду. Эта точка монтирования не сохранится после перезагрузки вашей системы.
```
$ sudo mount /dev/sdc1 /mnt/2g-new
```

Для постоянного монтирования добавьте информацию о разделе в файл fstab. Это можно сделать двумя способами: добавить имя устройства или значение UUID. Постоянное монтирование с использованием имени устройства:

```
# vi /etc/fstab
/dev/sdc1 /mnt/2g-new ext4 defaults 0 0
```
Постоянное монтирование с использованием значения UUID. Чтобы получить UUID раздела, используйте команду blkid.

```
$ sudo blkid
/dev/sdc1: UUID="d17e3c31-e2c9-4f11-809c-94a549bc43b7" TYPE="ext2" PARTUUID="8cc8f9e5-01"
/dev/sda1: UUID="d92fa769-e00f-4fd7-b6ed-ecf7224af7fa" TYPE="ext4" PARTUUID="eab59449-01"
/dev/sdc3: UUID="ca307aa4-0866-49b1-8184-004025789e63" TYPE="ext4" PARTUUID="8cc8f9e5-03"
/dev/sdc5: PARTUUID="8cc8f9e5-05"

# vi /etc/fstab

UUID=d17e3c31-e2c9-4f11-809c-94a549bc43b7 /mnt/2g-new ext4 defaults 0 0
То же самое было проверено с помощью команды df.

$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            969M     0  969M   0% /dev
tmpfs           200M  7.0M  193M   4% /run
/dev/sda1        20G   16G  3.0G  85% /
tmpfs           997M     0  997M   0% /dev/shm
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
tmpfs           997M     0  997M   0% /sys/fs/cgroup
tmpfs           200M   28K  200M   1% /run/user/121
tmpfs           200M   25M  176M  13% /run/user/1000
/dev/sdc1      1008M  1.3M  956M   1% /mnt/2g-new
```


## [[⬆]](#toc) <a name='14'>Дополнительные источники</a>
[Администрирование систем Linux. Разделы жестких дисков](https://rus-linux.net/MyLDP/BOOKS/LSA/ch05.html)
[Управление дисками и устройствами в Linux](https://serverspace.ru/support/help/upravlenie-blochnymi-ustrojstvami-linux/)




