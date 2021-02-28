# linux_pro_2021
# HW2
За основу взять Vagrantfile отсюда
```
https://github.com/erlong15/otus-linux
```
По методичке, пройдены все этапы. Выполнено задание со *, виртуалка собирается сразу с RAID6 путем добавления в Vagrantfile скрипта следующего содержания
```
mdadm --zero-superblock --force /dev/sd{b,c,d,e,f}
mdadm --create --verbose /dev/md0 --level=6 --raid-devices=5 /dev/sd{b,c,d,e,f}
```
В команде, создающей рейд, прописаны полные ключи команды mdadm, т.к. сокращения на машине выдавали ошибку.

# HW3
Так как четких критериев сдачи не прописано, описываю шаги выполненного задания в README с выводом.

## Уменьшние тома
### Создание временного тома и переноса туда данных
```
[root@lvm ~]# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
[root@lvm ~]# vgcreate vg_root /dev/sdb
  Volume group "vg_root" successfully created
[root@lvm ~]# lvcreate -n lv_root -l +100%FREE /dev/vg_root
  Logical volume "lv_root" created.
[root@lvm ~]# mkfs.xfs /dev/vg_root/lv_root
meta-data=/dev/vg_root/lv_root   isize=512    agcount=4, agsize=655104 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=2620416, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@lvm ~]# mount /dev/vg_root/lv_root /mnt
[root@lvm ~]# xfsdump -J - /dev/VolGroup00/LogVol00 | xfsrestore -J - /mnt

------------
xfsdump: dump size (non-dir files) : 715687944 bytes
xfsdump: dump complete: 10 seconds elapsed
xfsdump: Dump Status: SUCCESS
xfsrestore: restore complete: 10 seconds elapsed
xfsrestore: Restore Status: SUCCESS
[root@lvm ~]# for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
[root@lvm ~]# chroot /mnt/
[root@lvm /]# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
done
[root@lvm /]# cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g;
> s/.img//g"` --force; done

-----------
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***
[root@lvm boot]# vi /boot/grub2/grub.cfg
[root@lvm boot]# exit
exit
[root@lvm ~]# shutdown now -r
[root@lvm ~]# lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk 
├─sda1                    8:1    0    1M  0 part 
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part 
  ├─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
  └─VolGroup00-LogVol00 253:2    0 37.5G  0 lvm  
sdb                       8:16   0   10G  0 disk 
└─vg_root-lv_root       253:0    0   10G  0 lvm  /
sdc                       8:32   0    2G  0 disk 
sdd                       8:48   0    1G  0 disk 
sde                       8:64   0    1G  0 disk 
```
### Изменение исходного тома и возвращение туда рута
```
[root@lvm ~]# lvremove /dev/VolGroup00/LogVol00
Do you really want to remove active logical volume VolGroup00/LogVol00? [y/n]: y
  Logical volume "LogVol00" successfully removed
[root@lvm ~]# lvcreate -n VolGroup00/LogVol00 -L 8G /dev/VolGroup00
WARNING: xfs signature detected on /dev/VolGroup00/LogVol00 at offset 0. Wipe it? [y/n]: y
  Wiping xfs signature on /dev/VolGroup00/LogVol00.
  Logical volume "LogVol00" created.
[root@lvm ~]# mkfs.xfs /dev/VolGroup00/LogVol00
meta-data=/dev/VolGroup00/LogVol00 isize=512    agcount=4, agsize=524288 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=2097152, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@lvm ~]# mount /dev/VolGroup00/LogVol00 /mnt
[root@lvm ~]# xfsdump -J - /dev/vg_root/lv_root | xfsrestore -J - /mnt

----------
xfsdump: dump size (non-dir files) : 714342736 bytes
xfsdump: dump complete: 14 seconds elapsed
xfsdump: Dump Status: SUCCESS
xfsrestore: restore complete: 14 seconds elapsed
xfsrestore: Restore Status: SUCCESS
[root@lvm ~]# for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
[root@lvm ~]# chroot /mnt/
[root@lvm /]# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
done
[root@lvm /]# cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g;
> s/.img//g"` --force; done

----------
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***
```
## Не отходя от кассы, делаем mirror для /var
```
[root@lvm boot]# pvcreate /dev/sdc /dev/sdd
  Physical volume "/dev/sdc" successfully created.
  Physical volume "/dev/sdd" successfully created.
[root@lvm boot]# vgcreate vg_var /dev/sdc /dev/sdd
  Volume group "vg_var" successfully created
[root@lvm boot]#  lvcreate -L 950M -m1 -n lv_var vg_var
  Rounding up size to full physical extent 952.00 MiB
  Logical volume "lv_var" created.
[root@lvm boot]# mkfs.ext4 /dev/vg_var/lv_var
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
60928 inodes, 243712 blocks
12185 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=249561088
8 block groups
32768 blocks per group, 32768 fragments per group
7616 inodes per group
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

[root@lvm boot]# mount /dev/vg_var/lv_var /mnt
[root@lvm boot]# cp -aR /var/* /mnt/
[root@lvm boot]# rsync -avHPSAX /var/ /mnt/
sending incremental file list
./
.updated
            163 100%    0.00kB/s    0:00:00 (xfr#1, ir-chk=1028/1030)

sent 130,198 bytes  received 561 bytes  87,172.67 bytes/sec
total size is 125,494,687  speedup is 959.74
[root@lvm boot]# mkdir /tmp/oldvar && mv /var/* /tmp/oldvar
[root@lvm boot]# umount /mnt
[root@lvm boot]# mount /dev/vg_var/lv_var /var
[root@lvm boot]# echo "`blkid | grep var: | awk '{print $2}'` /var ext4 defaults 0 0" >> /etc/fstab
[root@lvm boot]# exit
exit
[root@lvm ~]# shutdown now -r
[root@lvm ~]#  lvremove /dev/vg_root/lv_root
Do you really want to remove active logical volume vg_root/lv_root? [y/n]: y
  Logical volume "lv_root" successfully removed
[root@lvm ~]#  vgremove /dev/vg_root
  Volume group "vg_root" successfully removed
[root@lvm ~]# pvremove /dev/sdb
  Labels on physical volume "/dev/sdb" successfully wiped.
```
## Выделяем том под /home и прописываем его в fstab
```
[root@lvm ~]#  lvcreate -n LogVol_Home -L 2G /dev/VolGroup00
  Logical volume "LogVol_Home" created.
[root@lvm ~]# mkfs.xfs /dev/VolGroup00/LogVol_Home
meta-data=/dev/VolGroup00/LogVol_Home isize=512    agcount=4, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@lvm ~]# mount /dev/VolGroup00/LogVol_Home /mnt/
[root@lvm ~]# cp -aR /home/* /mnt/ 
[root@lvm ~]# rm -rf /home/*
[root@lvm ~]# umount /mnt
[root@lvm ~]# mount /dev/VolGroup00/LogVol_Home /home/
[root@lvm ~]# echo "`blkid | grep Home | awk '{print $2}'` /home xfs defaults 0 0" >> /etc/fstab
```
## Наконец, упражняемся со снэпшотами
Создаем для примера 20 файлов
```
[root@lvm ~]# touch /home/file{1..20}
[root@lvm ~]# ls -a /home/
.   file1   file11  file13  file15  file17  file19  file20  file4  file6  file8  vagrant
..  file10  file12  file14  file16  file18  file2   file3   file5  file7  file9
```
Делаем снэпшот и сносим десяток файлов
```
[root@lvm ~]# lvcreate -L 100MB -s -n home_snap /dev/VolGroup00/LogVol_Home
  Rounding up size to full physical extent 128.00 MiB
  Logical volume "home_snap" created.
[root@lvm ~]# rm -f /home/file{11..20}
```
После чего восстанавливаемся из снэпшота
```
[root@lvm ~]# umount /home
[root@lvm ~]# lvconvert --merge /dev/VolGroup00/home_snap
  Merging of volume VolGroup00/home_snap started.
  VolGroup00/LogVol_Home: Merged: 100.00%
[root@lvm ~]# mount /home
[root@lvm ~]# ls -a /home/
.   file1   file11  file13  file15  file17  file19  file20  file4  file6  file8  vagrant
..  file10  file12  file14  file16  file18  file2   file3   file5  file7  file9
```

## Финальный вывод lsblk нужных томов
```
└─sda3                       8:3    0   39G  0 part 
  ├─VolGroup00-LogVol00    253:0    0    8G  0 lvm  /
  └─VolGroup00-LogVol_Home 253:7    0    2G  0 lvm  /home
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
```
# HW5
Задание вновь не предусматривает ничего кроме описания, поэтому прописываю в Readme. За основу взят Vagrantfile из ДЗ №3 (лежит в репе соответвующей папке).
## Несколькими способами попасть в систему без пароля
Всеми тремя способами терпим фиаско - первый и второй вываливают в кернел панику, третий просто вешает ВМ. По следам чата в слаке, находим занимателньое решение - выпилить аргументы 
```
console=tty0 console=ttyS0,115200n8
```
После этого все три способа прекрасно срабатывают, как и описано в методичке. Первым добавляем 
```
init=/bin/sh
```
и после 
```
mount -o remount,rw /
```
можно проверить работоспособность, например, запилив через echo файл с текстом. 
Способ 2, добавить в конец строчки linux 16 
```
rd.break
```
и после произвести манипуляции по переводу ФС в режим записи и смены пароля рута путем
```
# mount -o remount,rw /sysroot
# chroot /sysroot
# passwd root
# touch /.autorelabel
```
также работает только при удалении "консольных" аргументов. Проверку смены пароля произвести напрямую не удалось, т.к. вагрантовский плейбук напрямую бросает в пользователя lvm)Поэтому проверка осуществлялась по другому - через 
```
vagrant ssh
```
Ну и наконец также был првоерен способ 3, заключавшийся в добавлении
```
rw init=/sysroot/bin/sh
```
опять же с удалением "консольных" аргументов. Все сработало на ура.
## Переименовать VG
Вообщем то простое задание было здорово растянуто по времени из за невнимательности. Имя VG, как и все, христоматийно совпадало с примером. Сразу поставил nano на ВМ, т.к. борода еще не доросла до vim
```
# yum install nano
```
Меняем имя на OR, ибо так короче
```
# vgrename VolGroup00 OR
  Volume group "VolGroup00" successfully renamed to "OR"
```
Меняем в файлах 
```
# nano /etc/fstab 
# nano /etc/default/grub 
# nano /boot/grub2/grub.cfg 
```
имя VG с VolGroup00 на OR, пересоздаем образ initrd
```
# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
```
и...ВМ намертво зависает при перезагрузке после выбора ядра. И так три раза. Пока не выясняется, что в grub.cfg VG упоминается в трех местах, а я поменял в одном...fail. Переделываем, перезагружаемся и машина снова зависает. Грустим, ан всякий перезагружаем еще раз...и ВМ заводится. Щастье радость, VG переименована
```
# vgs
  VG #PV #LV #SN Attr   VSize   VFree
  OR   1   2   0 wz--n- <38.97g    0 
```
Можно наконец ехать дальше.
## Добавляем пингвинский модуль в initrd
На удивление, тут уже все пошло без казусов) Создаем директорию для своих модулей и едем туда
```
# mkdir /usr/lib/dracut/modules.d/01test
# cd /usr/lib/dracut/modules.d/01test/
```
Создаем пару файлов и заносим туда скрипты из методички
```
# echo > module-setup.sh
# nano module-setup.sh 
# echo > test.sh
# nano test.sh 
```
Пересобираем образ initrd
```
# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)

---------
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***

# dracut -f -v

---------
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***
```
проверяем, что модуль установлися
```
# lsinitrd -m /boot/initramfs-$(uname -r).img | grep test
test
```
так как уже наизусть знаем местоположение grub.cfg, прям там удаляем rghb и quiet из строки с linux 16 и решительно перезагружаемся! При перезугрузке видим
```
Hello! You are in dracut module!
 ___________________
< I'm dracut module >
 -------------------
   \
    \
        .--.
       |o_o |
       |:_/ |
      //   \ \
     (|     | )
    /'\_   _/`\
    \___)=(___/
continuing....
```
радуемся и бежим сдавать эту ДЗ, дабы успеть сделать еще парочку)