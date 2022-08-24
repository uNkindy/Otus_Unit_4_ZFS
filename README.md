### Домашнее задание №4 ZFS
#### 1. Тюнинг Vagrantfile (прилагается):
* Добавлено 3 диска IDE /dev/sd[b-d];
* Добавлено исполнение shell скрипта __setup_zfs.sh__;
* Гостевая ОС __almalinux/8__;
* Запуск гостевой ОС производится с правами root;
____
#### 2. Написан shell-скрипт для установки zfs (прилагается):
* Скрипт написан под установку RHEL-based distro (almalinux/8).
* Режим установки kABI-tracking kmod.
* Поправлены локали, определяющие язык и настройки кодировки.
* Добавлена проверка корректной правки локалей.
* Установка пакета wget.
____
#### 3. Определение алгоритма с наилучшим сжатием:
* Создал пул из 3 дисков __sdb__, __sdc__, __sdd__ с именем __raid_zfs__ и объединением в __raidz1__.
* Проверил корректность создания пула командой: 

[root@server ~]# zpool status  
  pool: raid_zfs  
 state: ONLINE  
config:  
  
	NAME        STATE     READ WRITE CKSUM  
	raid_zfs    ONLINE       0     0     0  
	  raidz1-0  ONLINE       0     0     0  
	    sdb     ONLINE       0     0     0  
	    sdc     ONLINE       0     0     0  
	    sdd     ONLINE       0     0     0  
  
  errors: No known data errors  

* Создал 4 файловые системы __/raid_zfs/gzip__, __/raid_zfs/lz4__, __/raid_zfs/lzjb__, __/raid_zfs/zle__
 
 [root@server ~]# mount -t zfs  
 raid_zfs on /raid_zfs type zfs (rw,seclabel,xattr,noacl)  
 raid_zfs/gzip on /raid_zfs/gzip type zfs (rw,seclabel,xattr,noacl)  
 raid_zfs/zle on /raid_zfs/zle type zfs (rw,seclabel,xattr,noacl)  
 raid_zfs/lzjb on /raid_zfs/lzjb type zfs (rw,seclabel,xattr,noacl)  
 raid_zfs/lz4 on /raid_zfs/lz4 type zfs (rw,seclabel,xattr,noacl)  
 
 * Скачал файл War_and_Peace.txt при помощи команды wget.

 [root@server ~]# ls -alh /root/War_and_Peace.txt   
-rw-r--r--. 1 root root 3.3M Aug  2 08:36 /root/War_and_Peace.txt  

 * Настроил компрессию соотвестсвенно названиям файловых систем командой __zfs compression=zle /raid_zfs/zle__.
 
[root@server ~]# zfs get compression  
NAME           PROPERTY     VALUE           SOURCE  
raid_zfs       compression  off             default  
raid_zfs/gzip  compression  gzip            local  
raid_zfs/lz4   compression  lz4             local  
raid_zfs/lzjb  compression  lzjb            local  
raid_zfs/zle   compression  zle             local  
 
 * Скопировал файл War_and_Peace.txt на 4 файловые системы с разной компрессией:
 
 [root@server ~]# zfs get  compressratio,compression  
NAME           PROPERTY       VALUE           SOURCE  
raid_zfs       compressratio  1.47x           -  
raid_zfs       compression    off             default  
raid_zfs/gzip  compressratio  2.67x           -  
raid_zfs/gzip  compression    gzip            local  
raid_zfs/lz4   compressratio  1.63x           -  
raid_zfs/lz4   compression    lz4             local  
raid_zfs/lzjb  compressratio  1.36x           -  
raid_zfs/lzjb  compression    lzjb            local  
raid_zfs/zle   compressratio  1.01x           -  
raid_zfs/zle   compression    zle             local  

__В результате самым эффективным методом сжатия текстовых файлов является gzip.__
