Прерываемая ВМ создана ранее
Во время просмотра вебинара:
Поставлен Docker Engine
Создана docker-сеть
Разворачиваю:
admin@avdonina-vm:~$ sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15
1c94a2f29db33bccb561cabbf33c3d0f4477aa2a5ccc2e050f6cc2bcba7be0f0
admin@avdonina-vm:~$ sudo docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS         PORTS                                       NAMES
1c94a2f29db3   postgres:15   "docker-entrypoint.s…"   10 seconds ago   Up 9 seconds   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   pg-server
admin@avdonina-vm:~$ sudo docker run -it --rm --network pg-net --name pg-client postgres:15 psql -h pg-server -U postgres
Password for user postgres: 
psql (15.5 (Debian 15.5-1.pgdg120+1))
Type "help" for help.
Создаю БД с таблицей:
postgres=> CREATE DATABASE work;
CREATE DATABASE
postgres=> SELECT current_database();
 current_database 
------------------
 postgres
(1 row)

postgres=> CREATE TABLE work1212 (id serial, hight int);
CREATE TABLE
postgres=> INSERT INTO work1212(hight) VALUES (100500);
INSERT 0 1
postgres=> INSERT INTO work1212(hight) VALUES (9100500);
INSERT 0 1
postgres=> SELECT * FROM work1212;
 id |  hight  
----+---------
  1 |  100500
  2 | 9100500
(2 rows)
postgres=> COMMIT
postgres-> \q
Подключаюсь по внешнему IP:
admin@avdonina-vm:~$ psql -p 5432 -U postgres -h 158.160.147.65 -d postgres -W
Password: 
psql (16.1 (Ubuntu 16.1-1.pgdg20.04+1), server 15.5 (Debian 15.5-1.pgdg120+1))
Type "help" for help.

postgres=> \q
admin@avdonina-vm:~$ docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS                                       NAMES
1c94a2f29db3   postgres:15   "docker-entrypoint.s…"   15 minutes ago   Up 15 minutes   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   pg-server
admin@avdonina-vm:~$ sudo docker stop 1c94a2f29db3   -  останавливаю
1c94a2f29db3
admin@avdonina-vm:~$ sudo docker rm 1c94a2f29db3    - удаляю контейнер
1c94a2f29db3
Создаю заново:
admin@avdonina-vm:~$ sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15
df0b3070328ae0008c7c81455ea1bdc74f2db1ddbd65eedc176472c0241d6c69
admin@avdonina-vm:~$ docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                                       NAMES
df0b3070328a   postgres:15   "docker-entrypoint.s…"   8 seconds ago   Up 7 seconds   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   pg-server
admin@avdonina-vm:~$ sudo docker run -it --rm --network pg-net --name pg-client postgres:15 psql -h pg-server -U postgres
Password for user postgres: 
psql (15.5 (Debian 15.5-1.pgdg120+1))
Type "help" for help.

postgres=> \dt
          List of relations
 Schema |   Name   | Type  |  Owner   
--------+----------+-------+----------
 public | work1212 | table | postgres
(1 row)
postgres=> SELECT * FROM work1212;
 id |  hight  
----+---------
  1 |  100500
  2 | 9100500
(2 rows)
Данные на месте, т.к. контейнер - это виртуализация. Данные хранятся на хостовой машине
