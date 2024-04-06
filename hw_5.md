```
admin@avdonina-vm:~$ sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet
-O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql-15

admin@avdonina-vm:~$ pg_lsclusters 
Ver Cluster Port Status Owner    Data directory              Log file 
15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log

admin@avdonina-vm:~$ sudo -u postgres pgbench -i -U postgres postgres 
dropping old tables... 
NOTICE:  table "pgbench_accounts" does not exist, skipping 
NOTICE:  table "pgbench_branches" does not exist, skipping 
NOTICE:  table "pgbench_history" does not exist, skipping 
NOTICE:  table "pgbench_tellers" does not exist, skipping 
creating tables... 
generating data (client-side)... 
100000 of 100000 tuples (100%) done (elapsed 0.07 s, remaining 0.00 s) 
vacuuming... 
creating primary keys... 
done in 1.19 s (drop tables 0.00 s, create tables 0.00 s, client-side generate 0.87 s, vacuum 0.03 s, primary keys 0.28 s)//создаю БД для сбора статистики

admin@avdonina-vm:~$ sudo -u postgres pgbench -c8 -P 6 -T 60 -U postgres postgres      
pgbench (15.6 (Ubuntu 15.6-1.pgdg20.04+1)) 
starting vacuum...end. 
progress: 6.0 s, 638.0 tps, lat 12.466 ms stddev 50.395, 0 failed 
progress: 12.0 s, 701.8 tps, lat 11.378 ms stddev 12.366, 0 failed 
progress: 18.0 s, 787.8 tps, lat 10.136 ms stddev 8.244, 0 failed 
progress: 24.0 s, 766.7 tps, lat 10.399 ms stddev 8.638, 0 failed 
progress: 30.0 s, 764.5 tps, lat 10.453 ms stddev 8.901, 0 failed 
progress: 36.0 s, 617.5 tps, lat 12.924 ms stddev 51.867, 0 failed 
progress: 42.0 s, 666.0 tps, lat 11.984 ms stddev 12.949, 0 failed 
progress: 48.0 s, 869.0 tps, lat 9.196 ms stddev 7.386, 0 failed 
progress: 54.0 s, 768.7 tps, lat 10.382 ms stddev 8.433, 0 failed 
progress: 60.0 s, 769.0 tps, lat 10.378 ms stddev 8.034, 0 failed 
transaction type: <builtin: TPC-B (sort of)> 
scaling factor: 1 
query mode: simple 
number of clients: 8 
number of threads: 1 
maximum number of tries: 1 
duration: 60 s 
number of transactions actually processed: 44102 
number of failed transactions: 0 (0.000%) 
latency average = 10.860 ms 
latency stddev = 22.834 ms 
initial connection time = 14.130 ms 
tps = 735.004288 (without initial connection time)

admin@avdonina-vm:~$ sudo mcedit /etc/postgresql/15/main/postgresql.conf //в этом файле изменила параметры кластера

dmin@avdonina-vm:~$ sudo systemctl restart postgresql@15-main.service //перезапустила кластер 
admin@avdonina-vm:~$ sudo systemctl status postgresql@15-main.service //проверила, что запустился         
● postgresql@15-main.service - PostgreSQL Cluster 15-main 
    Loaded: loaded (/lib/systemd/system/postgresql@.service; enabled-runtime; vendor preset: enabled) 
    Active: active (running) since Sun 2024-03-24 19:43:53 UTC; 16s ago 
   Process: 13729 ExecStart=/usr/bin/pg_ctlcluster --skip-systemctl-redirect 15-main start (code=exited, status=0/SUCCESS) 
  Main PID: 13755 (postgres) 
     Tasks: 6 (limit: 4631) 
    Memory: 46.7M 
    CGroup: /system.slice/system-postgresql.slice/postgresql@15-main.service 
            ├─13755 /usr/lib/postgresql/15/bin/postgres -D /var/lib/postgresql/15/main -c config_file=/etc/postgresql/15/main/postgresql.conf 
            ├─13756 postgres: 15/main: checkpointer 
            ├─13757 postgres: 15/main: background writer 
            ├─13759 postgres: 15/main: walwriter 
            ├─13760 postgres: 15/main: autovacuum launcher 
            └─13761 postgres: 15/main: logical replication launcher 

Mar 24 19:43:51 avdonina-vm systemd[1]: Starting PostgreSQL Cluster 15-main... 
Mar 24 19:43:53 avdonina-vm systemd[1]: Started PostgreSQL Cluster 15-main.

admin@avdonina-vm:~$ sudo -u postgres pgbench -c8 -P 6 -T 60 -U postgres postgres      
pgbench (15.6 (Ubuntu 15.6-1.pgdg20.04+1)) 
starting vacuum...end. 
progress: 6.0 s, 790.8 tps, lat 10.053 ms stddev 8.572, 0 failed 
progress: 12.0 s, 762.7 tps, lat 10.470 ms stddev 11.012, 0 failed 
progress: 18.0 s, 769.3 tps, lat 10.371 ms stddev 8.034, 0 failed 
progress: 24.0 s, 771.8 tps, lat 10.353 ms stddev 8.391, 0 failed 
progress: 30.0 s, 769.3 tps, lat 10.373 ms stddev 8.038, 0 failed 
progress: 36.0 s, 755.2 tps, lat 10.573 ms stddev 9.828, 0 failed 
progress: 42.0 s, 772.8 tps, lat 10.328 ms stddev 11.541, 0 failed 
progress: 48.0 s, 774.2 tps, lat 10.305 ms stddev 8.176, 0 failed 
progress: 54.0 s, 773.3 tps, lat 10.332 ms stddev 8.746, 0 failed 
progress: 60.0 s, 773.5 tps, lat 10.312 ms stddev 8.373, 0 failed 
transaction type: <builtin: TPC-B (sort of)> 
scaling factor: 1 
query mode: simple 
number of clients: 8 
number of threads: 1 
maximum number of tries: 1 
duration: 60 s 
number of transactions actually processed: 46286 
number of failed transactions: 0 (0.000%) 
latency average = 10.347 ms 
latency stddev = 9.149 ms 
initial connection time = 14.963 ms 
tps = 771.396538 (without initial connection time)

//в результате изменений параметров:
пропускная способность (tps) выровнялась и выросла на 5 %;
задержка (lat) выровнялась и на 150% уменьшилось стандартное отклонение задержки (latency stddev);
число обработанных в тесте транзакций выросло на 5%.
Потому что:
shared_buffers увеличили с 128MB до 1GB
min_wal_size увеличили c 80MB до 4GB
max_wal_size увеличили c 1GB до 16GB
следовательно тестирование происходило в более благоприятных условиях с т.з. выделенных ресурсов.

­postgres=# create table bla (a text); 
CREATE TABLE
postgres=# insert into bla(a) select generate_series(1,1000000);   
INSERT 0 1000000

admin@avdonina-vm:~$ sudo -u postgres psql                     
psql (15.6 (Ubuntu 15.6-1.pgdg20.04+1)) 
Type "help" for help. 

postgres=# SELECT schemaname, 
postgres-#        C.relname AS "relation", 
postgres-#        pg_size_pretty (pg_relation_size(C.oid)) as table, 
postgres-#        pg_size_pretty (pg_total_relation_size (C.oid)-pg_relation_size(C.oid)) as index, 
postgres-#        pg_size_pretty (pg_total_relation_size (C.oid)) as table_index, 
postgres-#        n_live_tup 
postgres-# FROM pg_class C 
postgres-# LEFT JOIN pg_namespace N ON (N.oid = C .relnamespace) 
postgres-# LEFT JOIN pg_stat_user_tables A ON C.relname = A.relname 
postgres-# WHERE nspname NOT IN ('pg_catalog', 'information_schema') 
postgres-# AND C.relkind <> 'i' 
postgres-# AND nspname !~ '^pg_toast' 
postgres-# ORDER BY pg_total_relation_size (C.oid) DESC 
postgres-# ; 
schemaname |     relation     |  table  |  index  | table_index | n_live_tup  
------------+------------------+---------+---------+-------------+------------ 
public     | bla              | 35 MB   | 48 kB   | 35 MB       |    1000000 
public     | pgbench_accounts | 13 MB   | 2240 kB | 16 MB       |     100000 
public     | pgbench_history  | 2384 kB | 32 kB   | 2416 kB     |      46286 
public     | pgbench_branches | 152 kB  | 48 kB   | 200 kB      |          1 
public     | pgbench_tellers  | 56 kB   | 48 kB   | 104 kB      |         10 
(5 rows)

//5 раз обновляю строки
postgres=# UPDATE bla set a = '@' || a;    // и другие символы


postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'bla';  //количество мертвых строчек в таблице и когда последний раз приходил автовакуум
relname | n_live_tup | n_dead_tup | ratio% |       last_autovacuum         
---------+------------+------------+--------+------------------------------ 
bla     |    1000000 |          0 |      0 | 2024-04-06 21:35:17.99789+00 
(1 row)


//обновила строки ещё 5 раз

relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum         
---------+------------+------------+--------+------------------------------- 
bla     |    1000000 |     999724 |     99 | 2024-04-06 21:56:26.658734+00 
(1 row)

//размер файла с таблицей после обновлений

schemaname |     relation     |  table  |  index  | table_index | n_live_tup  
------------+------------------+---------+---------+-------------+------------ 
public     | bla              | 208 MB  | 88 kB   | 208 MB      |    1000000 
public     | pgbench_accounts | 13 MB   | 2240 kB | 16 MB       |     100000 
public     | pgbench_history  | 2384 kB | 32 kB   | 2416 kB     |      46286 
public     | pgbench_branches | 152 kB  | 48 kB   | 200 kB      |          1 
public     | pgbench_tellers  | 56 kB   | 48 kB   | 104 kB      |         10

postgres=# ALTER TABLE bla SET (autovacuum_enabled = off);        
ALTER TABLE// отключила автовакуум

//10 раз обновила строки

//размер файла:
schemaname |     relation     |  table  |  index  | table_index | n_live_tup  
------------+------------------+---------+---------+-------------+------------ 
public     | bla              | 569 MB  | 176 kB  | 569 MB      |    1000000 
public     | pgbench_accounts | 13 MB   | 2240 kB | 16 MB       |     100000 
public     | pgbench_history  | 2384 kB | 32 kB   | 2416 kB     |      46286 
public     | pgbench_branches | 152 kB  | 48 kB   | 200 kB      |          1 
public     | pgbench_tellers  | 56 kB   | 48 kB   | 104 kB      |         10 
(5 rows)

relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum         
---------+------------+------------+--------+------------------------------- 
bla     |    1000000 |    9996192 |    999 | 2024-04-06 21:57:17.480982+00 
(1 row)

Размер файла кратно увеличился за счет хранения мертвых строк.



























