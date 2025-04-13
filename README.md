# otus_storage_systems
“Файловые системы и LVM-1” курса Administrator Linux.Professional

К работе добавлен файл для автоматического создания рейда с помощью Ansible.
Далее описание работы без учета автоматической сборки рейда, для закрепления знаний. 

Jobs

0. Создание дисков для ВМ
VBoxManage createmedium disk --filename disk1.vdi --size 1024 --format VDI
VBoxManage createmedium disk --filename disk2.vdi --size 1024 --format VDI
VBoxManage createmedium disk --filename disk3.vdi --size 1024 --format VDI
VBoxManage createmedium disk --filename disk4.vdi --size 1024 --format VDI
VBoxManage createmedium disk --filename disk5.vdi --size 1024 --format VDI


1. Проверка существующих дисков
vagrant@nginx:~$ sudo lshw -short | grep disk
/0/100/d/1    /dev/sdb   disk       1073MB VBOX HARDDISK
/0/100/d/2    /dev/sdc   disk       1073MB VBOX HARDDISK
/0/100/d/3    /dev/sdd   disk       1073MB VBOX HARDDISK
/0/100/d/4    /dev/sde   disk       1073MB VBOX HARDDISK
/0/100/d/5    /dev/sdf   disk       1073MB VBOX HARDDISK

2. Создание рейда:
sudo mdadm --create --verbose /dev/md0 -l 6 -n 5 /dev/sd{b,c,d,e,f}
mdadm: layout defaults to left-symmetric
mdadm: layout defaults to left-symmetric
mdadm: chunk size defaults to 512K
mdadm: size set to 1046528K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.

3. Проверка рейда
cat /proc/mdstat
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10] 
md0 : active raid6 sdf[4] sde[3] sdd[2] sdc[1] sdb[0]
      3139584 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/5] [UUUUU]
sudo mdadm -D /dev/md0
/dev/md0:
        Raid Level : raid6
    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       3       8       64        3      active sync   /dev/sde
       4       8       80        4      active sync   /dev/sdf
4. Фейлим одно блочное устройство, для имитации сбоя
mdadm /dev/md0 --fail /dev/sde
mdadm: set /dev/sde faulty in /dev/md0
5. Проверяем рейд
cat /proc/mdstat
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10] 
md0 : active raid6 sdf[4] sde[3](F) sdd[2] sdc[1] sdb[0]
      3139584 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/4] [UUU_U]
sudo mdadm -D /dev/md0
/dev/md0:
        Raid Level : raid6
    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       -       0        0        3      removed
       4       8       80        4      active sync   /dev/sdf

       3       8       64        -      faulty   /dev/sde
6. Удаляем сломанный диск
sudo mdadm /dev/md0 --remove /dev/sde
mdadm: hot removed /dev/sde from /dev/md0
7. Добавляем новый диск в рейд
sudo mdadm /dev/md0 --add /dev/sde
mdadm: added /dev/sde
8. Проверяем наш рейд
sudo mdadm -D /dev/md0
/dev/md0:
        Raid Level : raid6
    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       5       8       64        3      active sync   /dev/sde
       4       8       80        4      active sync   /dev/sdf
9. Создаем GPT таблицу
sudo parted -s /dev/md0 mklabel gpt
10. Создаем партиции 
sudo parted /dev/md0 mkpart primary ext4 0% 20%
sudo parted /dev/md0 mkpart primary ext4 20% 40%
sudo parted /dev/md0 mkpart primary ext4 40% 60%        
sudo parted /dev/md0 mkpart primary ext4 60% 80%        
sudo parted /dev/md0 mkpart primary ext4 80% 100%       
11. Создаем на этих партициях ФС
for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done
12. Монтируем по каталогам
sudo mkdir -p /raid/part{1,2,3,4,5}
sudo bash -c 'for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done'
















