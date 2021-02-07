# linux_pro_2021
### HW2
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