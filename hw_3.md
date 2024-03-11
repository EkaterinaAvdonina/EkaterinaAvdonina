/*устанавливаю Postgres v.15*/ admin@avdonina-vm:~$ sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql-15
admin@avdonina-vm:~$ sudo -u postgres pg_lsclusters //проверяю, что кластер запущен
5  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
admin@avdonina-vm:~$ sudo -u postgres psql //захожу из-под пользователя postgres в psql, создаю таблицу, выхожу
psql (15.6 (Ubuntu 15.6-1.pgdg20.04+1))
Type "help" for help.


postgres=#  create table test(c1 text);
CREATE TABLE
postgres=# insert into test values('1');
INSERT 0 1
postgres-# \q
admin@avdonina-vm:~$ sudo -u postgres pg_ctlcluster 15 main stop //останавливаю
Warning: stopping the cluster using pg_ctlcluster will mark the systemd unit as failed. Consider using systemctl:
  sudo systemctl stop postgresql@15-main
admin@avdonina-vm:~$ sudo apt update
admin@avdonina-vm:~$ sudo apt install parted  //устанавливаю Parted
admin@avdonina-vm:~$ sudo parted -l | grep Error 
Error: /dev/vdb: unrecognised disk label  //отсутствует схема разбиения
admin@avdonina-vm:~$ lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda    252:0    0  15G  0 disk 
├─vda1 252:1    0   1M  0 part 
└─vda2 252:2    0  15G  0 part /
vdb    252:16   0  10G  0 disk   //нашла вновь созданный диск
admin@avdonina-vm:~$ sudo parted /dev/vdb mklabel gpt //выбираю стандарт GPT
Warning: The existing disk label on /dev/vdb will be destroyed and all data on this disk will be lost. Do you
want to continue?
Yes/No? yes                                                               
Information: You may need to update /etc/fstab.

admin@avdonina-vm:~$ sudo parted -a opt /dev/vdb mkpart primary ext4 0% 100%  //создаю раздел, охватывающий весь диск
Information: You may need to update /etc/fstab.

admin@avdonina-vm:~$ lsblk                                                
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda    252:0    0  15G  0 disk 
├─vda1 252:1    0   1M  0 part 
└─vda2 252:2    0  15G  0 part /
vdb    252:16   0  10G  0 disk 
└─vdb1 252:17   0  10G  0 part    //доступен новый раздел
admin@avdonina-vm:~$ sudo mkfs.ext4 -L datapartition /dev/vdb1   //утилита mkfs.ext4 
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 2620928 4k blocks and 655360 inodes
Filesystem UUID: 17c81c46-9fc6-4b87-a229-041f1e1cd837
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 

admin@avdonina-vm:~$ sudo lsblk -o NAME,FSTYPE,LABEL,UUID,MOUNTPOINT   //проверяю, что применены различные методы
NAME   FSTYPE LABEL         UUID                                 MOUNTPOINT
vda                                                              
├─vda1                                                           
└─vda2 ext4                 be2c7c06-cc2b-4d4b-96c6-e3700932b129 /
vdb                                                              
└─vdb1 ext4   datapartition 17c81c46-9fc6-4b87-a229-041f1e1cd837
admin@avdonina-vm:~$ sudo mkdir -p /mnt/data
admin@avdonina-vm:~$ sudo mount -o defaults /dev/vdb1 /mnt/data //временное монтирование
admin@avdonina-vm:~$ sudo umount /dev/vdb1 //отмонтировала
admin@avdonina-vm:~$ sudo nano /etc/fstab //открываю файл
UUID=17c81c46-9fc6-4b87-a229-041f1e1cd837 /mnt/data       ext4    defaults          0       2  //добавила строку монтирования
admin@avdonina-vm:~$ sudo mount -a //монтирование всего 
admin@avdonina-vm:~$ df -hT //смотрю список список примонтированных файловых систем
Filesystem     Type      Size  Used Avail Use% Mounted on
udev           devtmpfs  1.9G     0  1.9G   0% /dev
tmpfs          tmpfs     392M  824K  392M   1% /run
/dev/vda2      ext4       15G  3.3G   11G  23% /
tmpfs          tmpfs     2.0G     0  2.0G   0% /dev/shm
tmpfs          tmpfs     5.0M     0  5.0M   0% /run/lock
tmpfs          tmpfs     2.0G     0  2.0G   0% /sys/fs/cgroup
tmpfs          tmpfs     392M     0  392M   0% /run/user/1000
/dev/vdb1      ext4      9.8G   24K  9.3G   1% /mnt/data
admin@avdonina-vm:~$ echo "success" | sudo tee /mnt/data/test_file
success
admin@avdonina-vm:~$ cat /mnt/data/test_file
success
admin@avdonina-vm:~$ sudo rm /mnt/data/test_file //удаляю файл
admin@avdonina-vm:~$ sudo reboot //перезагружаю
Connection to 158.160.149.186 closed by remote host.
Connection to 158.160.149.186 closed.
oem@asus:~/.ssh$ ping 158.160.149.186
--- 158.160.149.186 ping statistics ---
86 packets transmitted, 8 received, 90.6977% packet loss, time 86885ms
admin@avdonina-vm:~$ df -hT // убеждаюсь, что раздел автоматически примонтирован
Filesystem     Type      Size  Used Avail Use% Mounted on
udev           devtmpfs  1.9G     0  1.9G   0% /dev
tmpfs          tmpfs     392M  812K  392M   1% /run
/dev/vda2      ext4       15G  3.3G   11G  23% /
tmpfs          tmpfs     2.0G  1.1M  2.0G   1% /dev/shm
tmpfs          tmpfs     5.0M     0  5.0M   0% /run/lock
tmpfs          tmpfs     2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/vdb1      ext4      9.8G   24K  9.3G   1% /mnt/data
tmpfs          tmpfs     392M     0  392M   0% /run/user/1000
rtt min/avg/max/mdev = 20.884/22.361/28.522/2.370 ms
admin@avdonina-vm:~$ sudo chown -R postgres:postgres /mnt/data/  //сделала пользователя postgres владельцем /mnt/data 
admin@avdonina-vm:~$ sudo mv /var/lib/postgresql/15 /mnt/data //перенесла
admin@avdonina-vm:~$ sudo -u postgres pg_ctlcluster 15 main start
Error: /var/lib/postgresql/15/main is not accessible or does not exist
Кластер не запустился, т.к. /var/lib/postgresql/15/main после переноса перестала быть доступной
admin@avdonina-vm:~$ sudo cat /etc/postgresql/15/main/postgresql.conf | grep "/var/lib/postgresql/"
data_directory = '/var/lib/postgresql/15/main'          # use data in another directory // нашла параметр data_directory 
admin@avdonina-vm:~$ sudo nano /etc/postgresql/15/main/possutgresql.conf //открываю файл для редактирования
data_directory = '/mnt/data/15/main'            # use data in another directory //поменяла путь к БД
admin@avdonina-vm:~$ sudo -u postgres pg_ctlcluster 15 main start
Warning: the cluster will not be running as a systemd service. Consider using systemctl:
  sudo systemctl start postgresql@15-main //требуется запуск этой службы
Removed stale pid file.
admin@avdonina-vm:~$ sudo systemctl start postgresql@15-main // запустила
admin@avdonina-vm:~$ sudo -u postgres psql  //зашла через psql
psql (15.6 (Ubuntu 15.6-1.pgdg20.04+1))
Type "help" for help.

postgres=# SELECT table_name FROM information_schema.tables //получаю перечень таблиц базы данных
postgres-# WHERE table_schema NOT IN ('information_schema','pg_catalog');
 table_name 
------------
 test
(1 row)

postgres=# SELECT * FROM test;
 c1 
----
 1
(1 row)


































