//кластер создан
admin@avdonina-vm:~$ sudo -u postgres psql //захожу под польз. postgres
psql (15.6 (Ubuntu 15.6-1.pgdg20.04+1), server 14.11 (Ubuntu 14.11-1.pgdg20.04+1)) 
Type "help" for help. 

postgres=# CREATE DATABASE testdb; //создаю БД
CREATE DATABASE 
postgres=# \c testdb; 
psql (15.6 (Ubuntu 15.6-1.pgdg20.04+1), server 14.11 (Ubuntu 14.11-1.pgdg20.04+1)) 
You are now connected to database "testdb" as user "postgres". 

testdb=# CREATE SCHEMA testnm;//создаю схему  
CREATE SCHEMA 

testdb=# SET search_path TO testnm,"$user",public; //устанавливаю последовательность поиска
SET 
testdb=# SHOW search_path;                         
      search_path        
------------------------- 
testnm, "$user", public 
(1 row) 

testdb=# CREATE role readonly; //создаю роль только для чтения
CREATE ROLE 
testdb=# grant connect on DATABASE testdb to readonly; //передаю этой роли право на подключение к базе данных testdb
GRANT 
testdb=# grant usage on SCHEMA testnm to readonly; //передаю роли право на использование схемы testnm
GRANT 
testdb=# grant SELECT on all TABLES in SCHEMA testnm TO readonly; //передаю роли право на чтение из всех таблиц схемы testnm
GRANT 
testdb=# ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO readonly; // разрешение на чтение всех новых таблиц созданных в будущем
ALTER DEFAULT PRIVILEGES 
testdb=# ALTER default privileges in SCHEMA testnm GRANT SELECT ON TABLES to readonly; //// разрешение на чтение всех новых таблиц в схеме testnm
ALTER DEFAULT PRIVILEGES 
testdb=# CREATE USER testread with password 'test123'; //создаю пользователя с паролем
CREATE ROLE 
testdb=# grant readonly TO testread; //передаю права readonly пользователю
GRANT ROLE 

testdb=# CREATE TABLE testnm.t1(c1 integer); //создаю таблицу 
CREATE TABLE 
testdb=# INSERT INTO testnm.t1 VALUES(1);//наполняю значением  
INSERT 0 1 

testdb=# \q 
admin@avdonina-vm:~$ sudo -u postgres psql -h 127.0.0.1 -U testread -d testdb -W //захожу под пользователем testread
Password:  
psql (15.6 (Ubuntu 15.6-1.pgdg20.04+1), server 14.11 (Ubuntu 14.11-1.pgdg20.04+1)) 
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off) 
Type "help" for help. 

testdb=> SELECT * FROM testnm.t1; //просматриваю содержимое таблицы 1
c1  
---- 
 1 
(1 row) 

testdb=> CREATE TABLE t2(c1 integer); //создаю вторую таблицу
CREATE TABLE 
testdb=> INSERT INTO t2 VALUES(2); //наполняю данными
INSERT 0 1 
testdb=> \q 
admin@avdonina-vm:~$ sudo -u postgres psql 
psql (15.6 (Ubuntu 15.6-1.pgdg20.04+1), server 14.11 (Ubuntu 14.11-1.pgdg20.04+1)) 
Type "help" for help. 

postgres=# \c testdb; 
psql (15.6 (Ubuntu 15.6-1.pgdg20.04+1), server 14.11 (Ubuntu 14.11-1.pgdg20.04+1)) 
You are now connected to database "testdb" as user "postgres". 
testdb=# REVOKE CREATE on SCHEMA public FROM public; //отзываю права доступа на создание в схеме public
REVOKE 
testdb=# REVOKE ALL on DATABASE testdb FROM public; //отзываю все права к БД testdb в схеме public
REVOKE 
testdb=# \q 
admin@avdonina-vm:~$ sudo -u postgres psql -h 127.0.0.1 -U testread -d testdb -W //захожу под польз. testread
Password:  
psql (15.6 (Ubuntu 15.6-1.pgdg20.04+1), server 14.11 (Ubuntu 14.11-1.pgdg20.04+1)) 
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off) 
Type "help" for help. 

testdb=> CREATE TABLE t3(c1 integer); //пытаюсь создать таблицу 3 
ERROR:  permission denied for schema public //ошибка, т.к. ранее отозвала права
LINE 1: CREATE TABLE t3(c1 integer); 
                    ^ 
testdb=> 

