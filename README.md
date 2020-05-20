# Урок 3. Задание 1.
В репозитории находятся файлы [Vagrantfile](Vagrantfile) и README.

## Описание Vagrantfile:
1. В файле находится параметры вирт. машини;
2. Использовал созданную перемнную Virtual, для переноса HDD на другой диск;

## Описание задания:
1. Загрузка вирт. машины и обновление  ОС. virtual up , yum update -y;

2. Смотрим какие устройства у нас:
[vagrant@lvm /]$ lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
sda      8:0    0  40G  0 disk 
└─sda1   8:1    0  40G  0 part /
sdb      8:16   0  10G  0 disk 
sdc      8:32   0   2G  0 disk 
sdd      8:48   0   1G  0 disk 
sde      8:64   0   1G  0 disk 

3. Да
