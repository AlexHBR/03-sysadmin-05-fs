# Домашнее задание к занятию "3.5. Файловые системы"

1. Узнайте о [sparse](https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D0%B7%D1%80%D0%B5%D0%B6%D1%91%D0%BD%D0%BD%D1%8B%D0%B9_%D1%84%D0%B0%D0%B9%D0%BB) (разряженных) файлах.  
>Разрежённый файл (англ. sparse file) — файл, в котором последовательности нулевых байтов[1] заменены на информацию об этих последовательностях (список дыр).  
dd if=/dev/zero of=./sparse-file bs=1 count=0 seek=200G  
или  
truncate -s200G ./sparse-file  
преобразование обычного файла в разрежённый (выполнение поиска дыр и записи их расположения (смещений и длин) в метаданные файла):  
cp --sparse=always ./simple-file ./sparse-file  

2. Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?  
>hardlink это ссылка на тот же самый файл и имеет тот же inode нельзя поменять права у одного и того же объекта на разные.

3. Сделайте `vagrant destroy` на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:

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
![img.png](img.png)  

4. Используя `fdisk`, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.  
![img_1.png](img_1.png)  
5. Используя `sfdisk`, перенесите данную таблицу разделов на второй диск.  
![img_2.png](img_2.png)  
6. Соберите `mdadm` RAID1 на паре разделов 2 Гб.   
![img_3.png](img_3.png)  
7. Соберите `mdadm` RAID0 на второй паре маленьких разделов.   
![img_4.png](img_4.png)  
8. Создайте 2 независимых PV на получившихся md-устройствах.    
![img_5.png](img_5.png)  
9. Создайте общую volume-group на этих двух PV.  
![img_6.png](img_6.png)  
10. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.  
![img_7.png](img_7.png)  
11. Создайте `mkfs.ext4` ФС на получившемся LV.  
![img_8.png](img_8.png)  
12. Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.  
![img_9.png](img_9.png)  
13. Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.  
![img_10.png](img_10.png)  
14. Прикрепите вывод `lsblk`.  
![img_11.png](img_11.png)  
15. Протестируйте целостность файла:  

     ```bash
     root@vagrant:~# gzip -t /tmp/new/test.gz
     root@vagrant:~# echo $?
     0
     ```
![img_13.png](img_13.png)  
16. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.  
![img_14.png](img_14.png)  
17. Сделайте `--fail` на устройство в вашем RAID1 md.  
![img_15.png](img_15.png)  
18. Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.  
![img_16.png](img_16.png)  
19. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:  

     ```bash
     root@vagrant:~# gzip -t /tmp/new/test.gz
     root@vagrant:~# echo $?
     0
     ```
![img_17.png](img_17.png)  
20. Погасите тестовый хост, `vagrant destroy`.  
>Хост загашен.
 