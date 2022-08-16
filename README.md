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

