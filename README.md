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
