# Домашнее задание к занятию "3.5. Файловые системы"

## 1. Узнайте о [sparse](https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D0%B7%D1%80%D0%B5%D0%B6%D1%91%D0%BD%D0%BD%D1%8B%D0%B9_%D1%84%D0%B0%D0%B9%D0%BB) (разряженных) файлах.

### Решение

Читаем статью на Википедии, просвещаемся 


## 2. Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?

### Решение

Нет. Исходя из:

```
Linux каждый файл имеет уникальный идентификатор - индексный дескриптор (inode). 
Это число, которое однозначно идентифицирует файл в файловой системе. 
Жесткая ссылка и файл, для которой она создавалась имеют одинаковые inode. 
Поэтому жесткая ссылка имеет те же права доступа, владельца и время последней модификации, что и целевой файл. 
```
Различаются только имена файлов. Фактически жесткая ссылка это еще одно имя для файла. 


## 3. Сделайте `vagrant destroy` на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:

    ```bash
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

    Данная конфигурация создаст новую виртуальную машину с двумя дополнительными неразмеченными дисками по 2.5 Гб.
    
### Решение
    
  Выполняем команду  `vagrant destroy`
	Заменяем все содержимое Vagrantfile приложенным
	После запуска виртуальной машины командой `lsblk` проверяем диски
  
 ![img.png](https://github.com/nalevov/DO-Netology-3.5/blob/main/lsblk.png)
 
  
 ## 4. Используя `fdisk`, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.
 
 ### Решение
 
 Командой `sudo fdisk /dev/sdb` запускаем интерактивный режим, далее присваиваем номер и размер разделов
 
![img.png](https://github.com/nalevov/DO-Netology-3.5/blob/main/fdisc.png)


## 5. Используя `sfdisk`, перенесите данную таблицу разделов на второй диск.

### Решение

Выполняем команду `sudo sfdisk -d /dev/sdb | sudo sfdisk /dev/sdc`

![img.png](https://github.com/nalevov/DO-Netology-3.5/blob/main/sfdisk.png)

## 6. Соберите `mdadm` RAID1 на паре разделов 2 Гб.

### Решение 

Выполняем команду `sudo mdadm --create --verbose /dev/md1 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1`

Получаем 
```
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
mdadm: size set to 2094080K
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.
```

## 7. Соберите `mdadm` RAID0 на второй паре маленьких разделов.

### Решение

Выполняем команду `sudo mdadm --create --verbose /dev/md0 --level=0 --raid-devices=2 /dev/sdb2 /dev/sdc2`

Получаем 

```
mdadm: chunk size defaults to 512K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```

![img.png](https://github.com/nalevov/DO-Netology-3.5/blob/main/Mdadm.png)


## 8. Создайте 2 независимых PV на получившихся md-устройствах.

### Решение

Выполняем команду `sudo pvcreate /dev/md1 /dev/md0`

Получаем 

```
Physical volume "/dev/md1" successfully created.
Physical volume "/dev/md0" successfully created.
```

![img.png](https://github.com/nalevov/DO-Netology-3.5/blob/main/pvcreate.png)


## 9. Создайте общую volume-group на этих двух PV.

### Решение

Выполняем команду `vgcreate vg1 /dev/md1 /dev/md0`

Получаем 

```
Volume group "vg1" successfully created
```

![img.png](https://github.com/nalevov/DO-Netology-3.5/blob/main/vgcreate.png)


## 10. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.

### Решение

Выполняем команду `sudo lvcreate -L 100M vg1 /dev/md0`

Получаем 

```
Logical volume "lvol0" created.
```


## 11. Создайте `mkfs.ext4` ФС на получившемся LV.

### Решение

Выполняем команду `sudo  mkfs.ext4 /dev/vg1/lvol0`

Получаем 

```
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done
Writing inode tables: done
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done
```

## 12. Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.

### Решение

Создаем директорию `sudo mkdir /tmp/new` монтируем раздел `sudo mount /dev/vg1/lvol0 /tmp/new`


## 13. Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.

### Решение

Помещаем тестовы файл wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz

![img.png](https://github.com/nalevov/DO-Netology-3.5/blob/main/lvcreate.png)

## 14. Прикрепите вывод `lsblk`.

### Решение

![img.png](https://github.com/nalevov/DO-Netology-3.5/blob/main/lsblk.png)


## 15. Протестируйте целостность файла:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```
   
### Решение

![img.png](https://github.com/nalevov/DO-Netology-3.5/blob/main/%D0%A6%D0%B5%D0%BB%D0%BE%D1%81%D1%82%D0%BD%D0%BE%D1%81%D1%82%D1%8C%20%D1%84%D0%B0%D0%B9%D0%BB%D0%B0.png)


  

