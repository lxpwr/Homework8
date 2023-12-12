Что было сделано:
1. Попал 3 разными способами в однопользовательский режим Centos. Для этого на этапе загрузки, когда появилось меню выбора ОС, нажал "е" и попал в редактор загрузчика. Были испробованы все способы из методички:

   init=/bin/sh
   
   rd.break
   
   Замена пункта "ro" на "rw /init=sysroot/bin/sh"

   Первые два способа подключают ФС рута ОС в режиме только-чтение, ее нужно перемонтировать командой "mount -o remount,rw /". Во втором и третьем способе для входа в рут системы нужно делать смену на него командой "chroot /sysroot". Все три способа позволяют сменить пароль пользователя root - "passwd root" и загрузиться,используя свои учетные данные
2. Было выполнено переименование Volume Group LVM:
   vgrename VolGroup00 OtusRoot

   Чтобы система при перезагрузке смогла загрузиться, новое название было прописано в 3 файла:
     /etc/fstab 
     /etc/default/grub 
     /boot/grub2/grub.cfg
     Я сделал это с помощью редактора nano, который предварительно установил на ВМ. Одна правка в первом файле, 2 во втором, 3 - в третьем
   Пересоздал образ initrd - mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
   
   Загрузка системы прошла успешно:

   [vagrant@grubtest ~]$ lsblk
NAME                  MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT

sda                     8:0    0   40G  0 disk 

├─sda1                  8:1    0    1M  0 part 

├─sda2                  8:2    0    1G  0 part /boot

└─sda3                  8:3    0   39G  0 part 

  ├─OtusRoot-LogVol00 253:0    0 37.5G  0 lvm  /
  
  └─OtusRoot-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
  
sdb                     8:16   0   10G  0 disk 

sdc                     8:32   0    2G  0 disk 


sdd                     8:48   0    1G  0 disk 

sde                     8:64   0    1G  0 disk 

[vagrant@grubtest ~]$ vgs

  WARNING: Running as a non-root user. Functionality may be unavailable.

  /run/lvm/lvmetad.socket: access failed: Permission denied
  
  WARNING: Failed to connect to lvmetad. Falling back to device scanning.
  
  /dev/mapper/control: open failed: Permission denied
  
  Failure to communicate with kernel device-mapper driver.
  
  Incompatible libdevmapper 1.02.146-RHEL7 (2018-01-22) and kernel driver (unknown version).
  
[vagrant@grubtest ~]$ sudo -i

[root@grubtest ~]# vgs

  VG       #PV #LV #SN Attr   VSize   VFree
  
  OtusRoot   1   2   0 wz--n- <38.97g    0 

   
3. Создал модуль и добавил его в initrd:
   
   [root@grubtest ~]# mkdir /usr/lib/dracut/modules.d/pingwin
   
   [root@grubtest ~]# cd /usr/lib/dracut/modules.d/pingwin
   
   [root@grubtest pingwin]# nano module-setup.sh
   
   [root@grubtest pingwin]# nano pingwin.sh
   
   [root@grubtest pingwin]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
   
   (тут я допустил ошибку - посчитал, что цифры 01 необязательны в наименовании директории с модулем и у меня после сборки образа, мой модуль не отображался)
   
 [root@grubtest pingwin]# lsinitrd -m /boot/initramfs-$(uname -r).img | grep pingwin
 
   Вывод команды был пустым.
   
Я переименовал каталог с модулем:

[root@grubtest ~]# mv /usr/lib/dracut/modules.d/pingwin/ /usr/lib/dracut/modules.d/01pingwin

и пересобрал образ еще раз: dracut -f -v

После этого все пришло в норму:

[root@grubtest ~]# lsinitrd -m /boot/initramfs-$(uname -r).img | grep ping

pingwin

При перезагрузке, убрав опции, я увидел пингвина в текстовом формате. Все. :)
