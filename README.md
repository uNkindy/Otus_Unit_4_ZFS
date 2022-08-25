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
---

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
---
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
 
NAME           	PROPERTY       	VALUE           SOURCE  

raid_zfs       	compressratio  	1.47x           -  
raid_zfs       	compression    	off             default  
raid_zfs/gzip  	compressratio  	__2.67x__           -  
raid_zfs/gzip  	compression    	gzip            local  
raid_zfs/lz4   	compressratio  	1.63x           -  
raid_zfs/lz4   	compression    	lz4             local  
raid_zfs/lzjb  	compressratio  	1.36x           -  
raid_zfs/lzjb  	compression    	lzjb            local  
raid_zfs/zle   	compressratio  	1.01x           -  
raid_zfs/zle   	compression    	zle             local  

__В результате самым эффективным методом сжатия текстовых файлов является gzip.__
____

#### Определение настроек pool'а:
* Загрузил архив локально.
* Скопировал командой __scp__ архив на гостевую ОС.
* При помощи команды __zfs import__ собрал pool ZFS и проверил статус pool.

[root@server ~]# zpool import -d zpoolexport/ otus  
[root@server ~]# zpool status  
  pool: otus  
 state: ONLINE  
status: Some supported and requested features are not enabled on the pool.  
	The pool can still be used, but some features are unavailable.  
action: Enable all features using 'zpool upgrade'. Once this is done,  
	the pool may no longer be accessible by software that does not support  
	the features. See zpool-features(7) for details.  
config:  

	NAME                         STATE     READ WRITE CKSUM  
	otus                         ONLINE       0     0     0  
	  mirror-0                   ONLINE       0     0     0  
	    /root/zpoolexport/filea  ONLINE       0     0     0  
	    /root/zpoolexport/fileb  ONLINE       0     0     0  

errors: No known data errors  

* Собранный pool __otus__ имеет следующие настройки:
  * Тип pool __mirror-0__.
  * Размер хранилища __480M__.

[root@server ~]# zpool list  
NAME   SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT  
otus   480M  2.09M   478M        -         -     0%     0%  1.00x    ONLINE  -  

  * Значение recordsize 128K.

[root@server ~]# zpool list  
NAME   SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT  
otus   480M  2.09M   478M        -         -     0%     0%  1.00x    ONLINE  -  

  * Используемое сжатие __zle__.

[root@server ~]# zfs get compression  
NAME            PROPERTY     VALUE           SOURCE  
otus            compression  zle             local  
otus/hometask2  compression  zle             inherited from otus  

  * Используемая контрольная сумма __sha256__.

[root@server ~]# zfs get checksum  
NAME            PROPERTY  VALUE      SOURCE  
otus            checksum  sha256     local  
otus/hometask2  checksum  sha256     inherited from otus  

  * 


  
  

