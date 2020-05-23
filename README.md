# Урок 3. Задание 1.
В репозитории находятся файлы [Vagrantfile](Vagrantfile), README и файлы логирования утилиты script [time_lesson_3](time_lesson_3) [work_lesson_3](work_lesson_3).

## Описание Vagrantfile:
1. В файле находится параметры вирт. машини;
2. Использовал созданную перемнную Virtual, для переноса HDD на другой диск;

## Описание задания:
1. Загрузка вирт. машины и обновление  ОС. virtual up , yum update -y;
2. Смотрим какие устройства у нас;

[vagrant@lvm /]$ lsblk
`NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk
 ├─sda1                    8:1    0    1M  0 part`


├─sda2                    8:2    0    1G  0 part /boot


└─sda3                    8:3    0   39G  0 part 


├─VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  /

 
└─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
  
sdb                       8:16   0   10G  0 disk 

sdc                       8:32   0    2G  0 disk 

sdd                       8:48   0    1G  0 disk 

sde                       8:64   0    1G  0 disk


  3. Выполнение задание: Уменьшение тома под / до 8G;
  
* 3.1.  Создание pv, vg, lv на /dev/sdb;
  
Вывод команд: pvs, vgs, lvs:

PV         VG         Fmt  Attr PSize   PFree

/dev/sda3  VolGroup00 lvm2 a--  <38.97g     0 

/dev/sdb              lvm2 ---   10.00g 10.00g

VG         #PV #LV #SN Attr   VSize   VFree 

  VolGroup00   1   2   0 wz--n- <38.97g      0 
  
  vg_root      1   0   0 wz--n- <10.00g <10.00g
  
LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert

  LogVol00 VolGroup00 -wi-ao---- <37.47g 
  
  LogVol01 VolGroup00 -wi-ao----   1.50g 
  
  lv_root  vg_root    -wi-a----- <10.00g   
	
* 3.2. Создание фс, монтирование и перенос данных;

ФС:
meta-data=/dev/vg_root/lv_root   isize=512    agcount=4, agsize=655104 blks

         =                       sectsz=512   attr=2, projid32bit=1
	 
         =                       crc=1        finobt=0, sparse=0
	 
data     =                       bsize=4096   blocks=2620416, imaxpct=25

         =                       sunit=0      swidth=0 blks
	 
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1

log      =internal log           bsize=4096   blocks=2560, version=2

         =                       sectsz=512   sunit=0 blks, lazy-count=1
	 
realtime =none                   extsz=4096   blocks=0, rtextents=0


Перенос данных:
sudo xfsdump -J - /dev/VolGroup00/LogVol00 | sudo xfsrestore -J - /mnt
fsrestore: restore complete: 69 seconds elapsed
xfsrestore: Restore Status: SUCCESS

ll -ali /mnt:
total 12
      64 drwxr-xr-x. 20 root root  268 May 20 08:57 .
      64 dr-xr-xr-x. 20 root root  268 May 19 20:52 ..
    8214 lrwxrwxrwx.  1 root root    7 May 20 08:56 bin -> usr/bin
    7782 drwxr-xr-x.  2 root root    6 May 12  2018 boot
и так далее

3.3. Переконфигурация GRUB и обновление образа initrd. ;

Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-1127.8.2.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-1127.8.2.el7.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
Found CentOS Linux release 7.8.2003 (Core) on /dev/mapper/vg_root-lv_root
done

*** Created microcode section ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/ls' done ***

	3.4. Произвели замену на rd.lvm.lv=vg_root/lv_root в файле grub.cfg. Перезагрузили систему;

Вывод lsblk:
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk 
├─sda1                    8:1    0    1M  0 part 
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part 
  ├─VolGroup00-LogVol00 253:1    0 37.5G  0 lvm  
  └─VolGroup00-LogVol01 253:2    0  1.5G  0 lvm  [SWAP]
sdb                       8:16   0   10G  0 disk 
└─vg_root-lv_root       253:0    0   10G  0 lvm  /
sdc                       8:32   0    2G  0 disk 
sdd                       8:48   0    1G  0 disk 
sde                       8:64   0    1G  0 disk 

	3.5. Вывод lsblk, после перезагрузки;

[vagrant@lvm ~]$ lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk 
├─sda1                    8:1    0    1M  0 part 
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part 
  ├─VolGroup00-LogVol00 253:1    0 37.5G  0 lvm  
  └─VolGroup00-LogVol01 253:2    0  1.5G  0 lvm  [SWAP]
sdb                       8:16   0   10G  0 disk 
└─vg_root-lv_root       253:0    0   10G  0 lvm  /
sdc                       8:32   0    2G  0 disk 
sdd                       8:48   0    1G  0 disk 
sde                       8:64   0    1G  0 disk 

	3.6. Далее для уменьшения / тома, необходимо удалить LogVol00, что изменить размер на 8G. Проделываем тоже самое что 3.1, 3.2, 3.3;

lvremove /dev/VolGroup00/LogVol00

lvcreate -n VolGroup00/LogVol00 -L 8G /dev/VolGroup00

mkfs.xfs /dev/VolGroup00/LogVol00
mount /dev/VolGroup00/LogVol00 /mnt
xfsdump -J - /dev/vg_root/lv_root | xfsrestore -J - /mnt
и так далее по инструкции;

	3.7. Перезагрузка;

4. Работа с var;

	4.1. Создание для нового раздела var -  pv, vgvar, lvvar:
pvcreate /dev/sd{c,d}
Physical volume "/dev/sdc" successfully created.
Physical volume "/dev/sdd" successfully created.

vgcreate vg_var /dev/sd{c,d}
Volume group "vg_var" successfully created

lvcreate -L 950M -m1 -n lv_var vg_var
Rounding up size to full physical extent 952.00 MiB
Logical volume "lv_var" created.

	4.2. Создается ФС на lv_var, монтируется ФС в папку mnt. Копируются файлы из /var в mnt/var (рукурсивно и сохр. прав). После размонитируется /mnt, монтируется уже в lv_var в /var. Правим fstab. Перезагружаемся и удаляем pv где хранилось vg_root/lv_root, т.е. /dev/sdb. Правим fstab. Перезагружаемся и удаляем pv в котором  vg_root/lv_root, т.е. /dev/sdb.

Вывод lsblk:
[root@lvm ~]# lsblk
NAME                     MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                        8:0    0   40G  0 disk 
├─sda1                     8:1    0    1M  0 part 
├─sda2                     8:2    0    1G  0 part /boot
└─sda3                     8:3    0   39G  0 part 
  ├─VolGroup00-LogVol00  253:0    0    8G  0 lvm  /
  └─VolGroup00-LogVol01  253:1    0  1.5G  0 lvm  [SWAP]
sdb                        8:16   0   10G  0 disk 
sdc                        8:32   0    2G  0 disk 
├─vg_var-lv_var_rmeta_0  253:3    0    4M  0 lvm  
│ └─vg_var-lv_var        253:7    0  952M  0 lvm  /var
└─vg_var-lv_var_rimage_0 253:4    0  952M  0 lvm  
  └─vg_var-lv_var        253:7    0  952M  0 lvm  /var
sdd                        8:48   0    1G  0 disk 
├─vg_var-lv_var_rmeta_1  253:5    0    4M  0 lvm  
│ └─vg_var-lv_var        253:7    0  952M  0 lvm  /var
└─vg_var-lv_var_rimage_1 253:6    0  952M  0 lvm  
  └─vg_var-lv_var        253:7    0  952M  0 lvm  /var
sde                        8:64   0    1G  0 disk 

5. Работа с home;

	5.1. Повторяем действия 4 пункта, только раздел /home;

Вывод lsblk:
[root@lvm ~]# lsblk
NAME                       MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                          8:0    0   40G  0 disk 
├─sda1                       8:1    0    1M  0 part 
├─sda2                       8:2    0    1G  0 part /boot
└─sda3                       8:3    0   39G  0 part 
  ├─VolGroup00-LogVol00    253:0    0    8G  0 lvm  /
  ├─VolGroup00-LogVol01    253:1    0  1.5G  0 lvm  [SWAP]
  └─VolGroup00-LogVol_Home 253:7    0    2G  0 lvm  /home
sdb                          8:16   0   10G  0 disk 
sdc                          8:32   0    2G  0 disk 
├─vg_var-lv_var_rmeta_0    253:2    0    4M  0 lvm  
│ └─vg_var-lv_var          253:6    0  952M  0 lvm  /var
└─vg_var-lv_var_rimage_0   253:3    0  952M  0 lvm  
  └─vg_var-lv_var          253:6    0  952M  0 lvm  /var
sdd                          8:48   0    1G  0 disk 
├─vg_var-lv_var_rmeta_1    253:4    0    4M  0 lvm  
│ └─vg_var-lv_var          253:6    0  952M  0 lvm  /var
└─vg_var-lv_var_rimage_1   253:5    0  952M  0 lvm  
  └─vg_var-lv_var          253:6    0  952M  0 lvm  /var
sde                          8:64   0    1G

	5.2. Создем snapshot для /home. Удаляем файлы из фс и восстанавливаем фс до удаления файлов с помощью фс;

Вывод lvs со снапшотом:
[root@lvm home]# lvs
  LV          VG         Attr       LSize   Pool Origin      Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00    VolGroup00 -wi-ao----   8.00g                                                         
  LogVol01    VolGroup00 -wi-ao----   1.50g                                                         
  LogVol_Home VolGroup00 owi-aos---   2.00g                                                         
  home_snap   VolGroup00 swi-a-s--- 128.00m      LogVol_Home 0.00                                   
  lv_var      vg_var     rwi-aor--- 952.00m                                         100.00

Вывод lsblk без снапшоты:
[root@lvm home]# lsblk
NAME                       MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                          8:0    0   40G  0 disk 
├─sda1                       8:1    0    1M  0 part 
├─sda2                       8:2    0    1G  0 part /boot
└─sda3                       8:3    0   39G  0 part 
  ├─VolGroup00-LogVol00    253:0    0    8G  0 lvm  /
  ├─VolGroup00-LogVol01    253:1    0  1.5G  0 lvm  [SWAP]
  └─VolGroup00-LogVol_Home 253:7    0    2G  0 lvm  /home
sdb                          8:16   0   10G  0 disk 

 
