# Compare the performance postgres and Aurora on Amazon Cloud



## 1.Login postgres with psql on my virtual linux machine



```she

postgres@pg01:/$ psql -h rds-fp-hypocloud-dev-app.c9ppab0cbd5z.us-east-1.rds.amazonaws.com -U hypocloud -d postgres -W
Password for user hypocloud:
psql (10.6 (Ubuntu 10.6-0ubuntu0.18.10.1), server 10.3)
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 25                                                                             6, compression: off)
Type "help" for help.

postgres=>

```



## 2.Create a user named pgbench



```sql


postgres=> CREATE ROLE pgbench LOGIN PASSWORD 'fWnv4q^wTv5orCwZ12s$nry('
postgres->   NOSUPERUSER
postgres->   INHERIT
postgres->   CREATEDB
postgres->   CREATEROLE
postgres->   NOREPLICATION
postgres->   VALID UNTIL 'infinity';
CREATE ROLE
postgres=> \q

```

Login out

## 3.Login with user pgbench and create a database named pgbench

```sql

postgres@pg01:/$ psql -h rds-fp-hypocloud-dev-app.c9ppab0cbd5z.us-east-1.rds.amazonaws.com -U pgbench -d postgres -W
Password for user pgbench:
psql (10.6 (Ubuntu 10.6-0ubuntu0.18.10.1), server 10.3)
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

postgres=> create database pgbench;
CREATE DATABASE

postgres=> \c pgbench
Password for user pgbench:
psql (10.6 (Ubuntu 10.6-0ubuntu0.18.10.1), server 10.3)
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
You are now connected to database "pgbench" as user "pgbench".
pgbench=>

pgbench=> \dt
Did not find any relations.

```



## 4.Connect rds for postgres host and init database pgbench

```shell
postgres@pg01:/home/pgbench_test$ pgbench -h rds-fp-hypocloud-dev-app.c9ppab0cbd5z.us-east-1.rds.amazonaws.com -U pgbench -i
Password:
NOTICE:  table "pgbench_history" does not exist, skipping
NOTICE:  table "pgbench_tellers" does not exist, skipping
NOTICE:  table "pgbench_accounts" does not exist, skipping
NOTICE:  table "pgbench_branches" does not exist, skipping
creating tables...
100000 of 100000 tuples (100%) done (elapsed 1.12 s, remaining 0.00 s)
vacuum...
set primary keys...
done.

```



```sql
pgbench=> select count(1) as cnt,'pgbench_accounts' as tname from pgbench_accounts
pgbench->  union all
pgbench->  select count(1) as cnt,'pgbench_branches' as tname from pgbench_branches
pgbench->  union all
pgbench->  select count(1) as cnt,'pgbench_history' as tname from pgbench_history
pgbench->  union all
pgbench->  select count(1) as cnt,'pgbench_tellers' as tname from pgbench_tellers ;
  cnt   |      tname
--------+------------------
 100000 | pgbench_accounts
      1 | pgbench_branches
      0 | pgbench_history
     10 | pgbench_tellers
(4 rows)

```



## 5.pgbench --help

```sh
postgres@pg01:/$ pgbench --help
pgbench is a benchmarking tool for PostgreSQL.

Usage:
  pgbench [OPTION]... [DBNAME]

Initialization options:
  -i, --initialize         invokes initialization mode
  -F, --fillfactor=NUM     set fill factor
  -n, --no-vacuum          do not run VACUUM after initialization
  -q, --quiet              quiet logging (one message each 5 seconds)
  -s, --scale=NUM          scaling factor
  --foreign-keys           create foreign key constraints between tables
  --index-tablespace=TABLESPACE
                           create indexes in the specified tablespace
  --tablespace=TABLESPACE  create tables in the specified tablespace
  --unlogged-tables        create tables as unlogged tables

Options to select what to run:
  -b, --builtin=NAME[@W]   add builtin script NAME weighted at W (default: 1)
                           (use "-b list" to list available scripts)
  -f, --file=FILENAME[@W]  add script FILENAME weighted at W (default: 1)
  -N, --skip-some-updates  skip updates of pgbench_tellers and pgbench_branches
                           (same as "-b simple-update")
  -S, --select-only        perform SELECT-only transactions
                           (same as "-b select-only")

Benchmarking options:
  -c, --client=NUM         number of concurrent database clients (default: 1)
  -C, --connect            establish new connection for each transaction
  -D, --define=VARNAME=VALUE
                           define variable for use by custom script
  -j, --jobs=NUM           number of threads (default: 1)
  -l, --log                write transaction times to log file
  -L, --latency-limit=NUM  count transactions lasting more than NUM ms as late
  -M, --protocol=simple|extended|prepared
                           protocol for submitting queries (default: simple)
  -n, --no-vacuum          do not run VACUUM before tests
  -P, --progress=NUM       show thread progress report every NUM seconds
  -r, --report-latencies   report average latency per command
  -R, --rate=NUM           target rate in transactions per second
  -s, --scale=NUM          report this scale factor in output
  -t, --transactions=NUM   number of transactions each client runs (default: 10)
  -T, --time=NUM           duration of benchmark test in seconds
  -v, --vacuum-all         vacuum all four standard tables before tests
  --aggregate-interval=NUM aggregate data over NUM seconds
  --log-prefix=PREFIX      prefix for transaction time log file
                           (default: "pgbench_log")
  --progress-timestamp     use Unix epoch timestamps for progress
  --sampling-rate=NUM      fraction of transactions to log (e.g., 0.01 for 1%)

Common options:
  -d, --debug              print debugging output
  -h, --host=HOSTNAME      database server host or socket directory
  -p, --port=PORT          database server port number
  -U, --username=USERNAME  connect as specified database user
  -V, --version            output version information, then exit
  -?, --help               show this help, then exit


```



## 6.Show the basic info of my postgres instance 

​      

| Paramaters              | Paramaters   Value    |
| ----------------------- | --------------------- |
| Instance class          | db.t2.micro           |
| Postgres Engine version | 10.3                  |
| vCPU                    | 1                     |
| RAM                     | 1GB                   |
| Storage type            | General Purpose (SSD) |
| Storage                 | 20 GiB                |
| License model           | Postgresql License    |

![1.rds_postgres_basic_info](D:/learn_git/postgres/images/20181228/1.rds_postgres_basic_info.jpg)



Max connections:

```sql
pgbench=> show max_connections;
 max_connections
-----------------
 87
(1 row)

```



Network:

pgbench location: China(Shenzhen)

rds postgres location:America(Virginia)



## 7.Test cases rds postgres instance

  



| number   of clients                           | number   of threads | number   of transactions actually processed                  | latency   average(ms) | including   tps | excluding   tps |
| --------------------------------------------- | ------------------- | ------------------------------------------------------------ | --------------------- | --------------- | --------------- |
| 1                                             | 1                   | 13                                                           | 1601.42               | 0.624446        | 0.666016        |
| 15                                            | 1                   | 15                                                           | 27694.417             | 0.541625        | 0.569207        |
| 30                                            | 1                   | 30                                                           | 54092.917             | 0.554601        | 0.568656        |
| 45                                            | 1                   | 45                                                           | 83282.876             | 0.540327        | 0.549439        |
| 50                                            | 1                   | 50                                                           | 90037.798             | 0.555322        | 0.56374         |
| 65                                            | 1                   | 65                                                           | 118146.069            | 0.550166        | 0.556602        |
| 80                                            | 1                   | FATAL: remaining connection slots are reserved for   non-replication superuser and rds_superuser connections |                       |                 |                 |
| 50                                            | 5                   | 63                                                           | 32920.327             | 1.518818        | 1.568954        |
| 50                                            | 20                  | 87                                                           | 23773.666             | 2.103167        | 2.175233        |
| 50                                            | 35                  | 90                                                           | 23125.404             | 2.162124        | 2.237998        |
| 50                                            | 50                  | 84                                                           | 24786.412             | 2.017234        | 2.092387        |
|                                               |                     |                                                              |                       |                 |                 |
| basic rules:number of clients<max_connections |                     |                                                              |                       |                 |                 |
| number of clients>number of threads           |                     |                                                              |                       |                 |                 |



### 7.1 1 client,1 thread

```shell
postgres@pg01:/home/pgbench_test/rds_postgres$ nohup pgbench -h rds-fp-hypocloud-dev-app.c9ppab0cbd5z.us-east-1.rds.amazonaws.com -U pgbench -c 1 -j 1 -T 20 -r pgbench > file2.out 2>&1
Password:
postgres@pg01:/home/pgbench_test/rds_postgres$ nohup pgbench -h rds-fp-hypocloud-dev-app.c9ppab0cbd5z.us-east-1.rds.amazonaws.com -U pgbench -c 1 -T 20 -r pgbench > postgres_1_1.out 2>&1
Password:
postgres@pg01:/home/pgbench_test/rds_postgres$ cat postgres_1_1.out
nohup: ignoring input
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 1
number of threads: 1
duration: 20 s
number of transactions actually processed: 13
latency average = 1601.420 ms
tps = 0.624446 (including connections establishing)
tps = 0.666016 (excluding connections establishing)
script statistics:
 - statement latencies in milliseconds:
         0.011  \set aid random(1, 100000 * :scale)
         0.002  \set bid random(1, 1 * :scale)
         0.002  \set tid random(1, 10 * :scale)
         0.002  \set delta random(-5000, 5000)
       213.795  BEGIN;
       214.131  UPDATE pgbench_accounts SET abalance = abalance + :delta WHERE aid = :aid;
       215.174  SELECT abalance FROM pgbench_accounts WHERE aid = :aid;
       214.356  UPDATE pgbench_tellers SET tbalance = tbalance + :delta WHERE tid = :tid;
       214.694  UPDATE pgbench_branches SET bbalance = bbalance + :delta WHERE bid = :bid;
       214.306  INSERT INTO pgbench_history (tid, bid, aid, delta, mtime) VALUES (:tid, :bid, :aid, :delta, CURRENT_TIMESTAMP);
       214.961  END;

```



### 7.2 remaing connection slots are reserved for superuser

remaining connection slots are reserved for non-replication superuser and rds_superuser connections

```shel
postgres@pg01:/home/pgbench_test/rds_postgres$ nohup pgbench -h rds-fp-hypocloud-dev-app.c9ppab0cbd5z.us-east-1.rds.amazonaws.com -U pgbench -c 80 -j 1 -T 20 -r pgbench > postgres_80_1.out 2>&1
Password:
postgres@pg01:/home/pgbench_test/rds_postgres$ cat postgres_80_1.out
nohup: ignoring input
starting vacuum...end.
connection to database "pgbench" failed:
FATAL:  remaining connection slots are reserved for non-replication superuser and rds_superuser connections
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 80
number of threads: 1
duration: 20 s
number of transactions actually processed: 0

```



### 7.3 50 clients,5 threads

```shell
postgres@pg01:/home/pgbench_test/rds_postgres$ nohup pgbench -h rds-fp-hypocloud-dev-app.c9ppab0cbd5z.us-east-1.rds.amazonaws.com -U pgbench -c 50 -j 5 -T 20 -r pgbench > postgres_50_5.out 2>&1
Password:
postgres@pg01:/home/pgbench_test/rds_postgres$ cat postgres_50_5.out
nohup: ignoring input
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 5
duration: 20 s
number of transactions actually processed: 63
latency average = 32920.327 ms
tps = 1.518818 (including connections establishing)
tps = 1.568954 (excluding connections establishing)
script statistics:
 - statement latencies in milliseconds:
         0.009  \set aid random(1, 100000 * :scale)
         0.004  \set bid random(1, 1 * :scale)
         0.003  \set tid random(1, 10 * :scale)
         0.001  \set delta random(-5000, 5000)
       216.150  BEGIN;
       216.748  UPDATE pgbench_accounts SET abalance = abalance + :delta WHERE aid = :aid;
       215.578  SELECT abalance FROM pgbench_accounts WHERE aid = :aid;
      9796.660  UPDATE pgbench_tellers SET tbalance = tbalance + :delta WHERE tid = :tid;
      3121.717  UPDATE pgbench_branches SET bbalance = bbalance + :delta WHERE bid = :bid;
       215.182  INSERT INTO pgbench_history (tid, bid, aid, delta, mtime) VALUES (:tid, :bid, :aid, :delta, CURRENT_TIMESTAMP);
       216.430  END;

```



### 7.4 50 clients,56 threads

```shell
postgres@pg01:/home/pgbench_test/rds_postgres$ nohup pgbench -h rds-fp-hypocloud-dev-app.c9ppab0cbd5z.us-east-1.rds.amazonaws.com -U pgbench -c 50 -j 56 -T 20 -r pgbench > postgres_50_56.out 2>&1
Password:
postgres@pg01:/home/pgbench_test/rds_postgres$ cat postgres_50_56.out
nohup: ignoring input
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 50
duration: 20 s
number of transactions actually processed: 90
latency average = 22906.691 ms
tps = 2.182768 (including connections establishing)
tps = 2.264614 (excluding connections establishing)
script statistics:
 - statement latencies in milliseconds:
         0.010  \set aid random(1, 100000 * :scale)
         0.004  \set bid random(1, 1 * :scale)
         0.005  \set tid random(1, 10 * :scale)
         0.002  \set delta random(-5000, 5000)
       214.548  BEGIN;
       215.553  UPDATE pgbench_accounts SET abalance = abalance + :delta WHERE aid = :aid;
       215.016  SELECT abalance FROM pgbench_accounts WHERE aid = :aid;
     12512.945  UPDATE pgbench_tellers SET tbalance = tbalance + :delta WHERE tid = :tid;
      2614.245  UPDATE pgbench_branches SET bbalance = bbalance + :delta WHERE bid = :bid;
       214.832  INSERT INTO pgbench_history (tid, bid, aid, delta, mtime) VALUES (:tid, :bid, :aid, :delta, CURRENT_TIMESTAMP);
       216.340  END;

```





## 8.Show the basic info of my Aurora for postgres instance 

​      

| Paramaters              | Paramaters   Value |
| ----------------------- | ------------------ |
| Instance class          | db.r4.large        |
| Postgres Engine version | 10.5               |
| vCPU                    | 2                  |
| RAM                     | 15.25 GB           |
| Storage type            | DB cluster         |
| Storage                 | 50G+ ,Auto extend  |
| License model           | Postgresql License |

![2.aurora_postgres_basic_info](D:/learn_git/postgres/images/20181228/2.aurora_postgres_basic_info.jpg)



Max connections:

```sql

pgbench=> show max_connections;
 max_connections
-----------------
 1660
(1 row)

```



Network:

pgbench location: China(Shenzhen)

rds postgres location:America(Virginia)



## 9.Test cases Aurora for postgres instance

  



| number   of clients                           | number   of threads | number   of transactions actually processed                  | latency   average(ms) | including   tps | excluding   tps |
| --------------------------------------------- | ------------------- | ------------------------------------------------------------ | --------------------- | --------------- | --------------- |
| 1                                             | 1                   | 13                                                           | 1601.42               | 0.624446        | 0.666016        |
| 15                                            | 1                   | 15                                                           | 27694.417             | 0.541625        | 0.569207        |
| 30                                            | 1                   | 30                                                           | 54092.917             | 0.554601        | 0.568656        |
| 45                                            | 1                   | 45                                                           | 83282.876             | 0.540327        | 0.549439        |
| 50                                            | 1                   | 50                                                           | 90037.798             | 0.555322        | 0.56374         |
| 65                                            | 1                   | 65                                                           | 118146.069            | 0.550166        | 0.556602        |
| 80                                            | 1                   | FATAL: remaining connection slots are reserved for   non-replication superuser and rds_superuser connections |                       |                 |                 |
| 50                                            | 5                   | 63                                                           | 32920.327             | 1.518818        | 1.568954        |
| 50                                            | 20                  | 87                                                           | 23773.666             | 2.103167        | 2.175233        |
| 50                                            | 35                  | 90                                                           | 23125.404             | 2.162124        | 2.237998        |
| 50                                            | 50                  | 84                                                           | 24786.412             | 2.017234        | 2.092387        |
|                                               |                     |                                                              |                       |                 |                 |
| basic rules:number of clients<max_connections |                     |                                                              |                       |                 |                 |
| number of clients>number of threads           |                     |                                                              |                       |                 |                 |



### 9.1 1 client,1 thread

```shell
postgres@pg01:/home/pgbench_test/aurora_postgres$ nohup pgbench -h rds-fp-hypocloud-qa-app.c9ppab0cbd5z.us-east-1.rds.amazonaws.com -U pgbench -c 1 -j 1 -T 20 -r pgbench > aurora_1_1.out 2>&1
Password:
postgres@pg01:/home/pgbench_test/aurora_postgres$ cat aurora_1_1.out
nohup: ignoring input
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 1
number of threads: 1
duration: 20 s
number of transactions actually processed: 13
latency average = 1607.361 ms
tps = 0.622138 (including connections establishing)
tps = 0.663219 (excluding connections establishing)
script statistics:
 - statement latencies in milliseconds:
         0.014  \set aid random(1, 100000 * :scale)
         0.027  \set bid random(1, 1 * :scale)
         0.004  \set tid random(1, 10 * :scale)
         0.001  \set delta random(-5000, 5000)
       214.151  BEGIN;
       215.163  UPDATE pgbench_accounts SET abalance = abalance + :delta WHERE aid = :aid;
       215.430  SELECT abalance FROM pgbench_accounts WHERE aid = :aid;
       215.154  UPDATE pgbench_tellers SET tbalance = tbalance + :delta WHERE tid = :tid;
       215.459  UPDATE pgbench_branches SET bbalance = bbalance + :delta WHERE bid = :bid;
       214.883  INSERT INTO pgbench_history (tid, bid, aid, delta, mtime) VALUES (:tid, :bid, :aid, :delta, CURRENT_TIMESTAMP);
       217.487  END;


```



### 9.2 remaing connection slots are reserved for superuser

remaining connection slots are reserved for non-replication superuser and rds_superuser connections

```shel
postgres@pg01:/home/pgbench_test/rds_postgres$ nohup pgbench -h rds-fp-hypocloud-dev-app.c9ppab0cbd5z.us-east-1.rds.amazonaws.com -U pgbench -c 80 -j 1 -T 20 -r pgbench > postgres_80_1.out 2>&1
Password:
postgres@pg01:/home/pgbench_test/rds_postgres$ cat postgres_80_1.out
nohup: ignoring input
starting vacuum...end.
connection to database "pgbench" failed:
FATAL:  remaining connection slots are reserved for non-replication superuser and rds_superuser connections
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 80
number of threads: 1
duration: 20 s
number of transactions actually processed: 0

```



### 9.3 50 clients,5 threads

```shell
postgres@pg01:/home/pgbench_test/rds_postgres$ nohup pgbench -h rds-fp-hypocloud-dev-app.c9ppab0cbd5z.us-east-1.rds.amazonaws.com -U pgbench -c 50 -j 5 -T 20 -r pgbench > postgres_50_5.out 2>&1
Password:
postgres@pg01:/home/pgbench_test/rds_postgres$ cat postgres_50_5.out
nohup: ignoring input
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 5
duration: 20 s
number of transactions actually processed: 63
latency average = 32920.327 ms
tps = 1.518818 (including connections establishing)
tps = 1.568954 (excluding connections establishing)
script statistics:
 - statement latencies in milliseconds:
         0.009  \set aid random(1, 100000 * :scale)
         0.004  \set bid random(1, 1 * :scale)
         0.003  \set tid random(1, 10 * :scale)
         0.001  \set delta random(-5000, 5000)
       216.150  BEGIN;
       216.748  UPDATE pgbench_accounts SET abalance = abalance + :delta WHERE aid = :aid;
       215.578  SELECT abalance FROM pgbench_accounts WHERE aid = :aid;
      9796.660  UPDATE pgbench_tellers SET tbalance = tbalance + :delta WHERE tid = :tid;
      3121.717  UPDATE pgbench_branches SET bbalance = bbalance + :delta WHERE bid = :bid;
       215.182  INSERT INTO pgbench_history (tid, bid, aid, delta, mtime) VALUES (:tid, :bid, :aid, :delta, CURRENT_TIMESTAMP);
       216.430  END;

```



### 9.4 50 clients,56 threads

```shell
postgres@pg01:/home/pgbench_test/rds_postgres$ nohup pgbench -h rds-fp-hypocloud-dev-app.c9ppab0cbd5z.us-east-1.rds.amazonaws.com -U pgbench -c 50 -j 56 -T 20 -r pgbench > postgres_50_56.out 2>&1
Password:
postgres@pg01:/home/pgbench_test/rds_postgres$ cat postgres_50_56.out
nohup: ignoring input
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 50
duration: 20 s
number of transactions actually processed: 90
latency average = 22906.691 ms
tps = 2.182768 (including connections establishing)
tps = 2.264614 (excluding connections establishing)
script statistics:
 - statement latencies in milliseconds:
         0.010  \set aid random(1, 100000 * :scale)
         0.004  \set bid random(1, 1 * :scale)
         0.005  \set tid random(1, 10 * :scale)
         0.002  \set delta random(-5000, 5000)
       214.548  BEGIN;
       215.553  UPDATE pgbench_accounts SET abalance = abalance + :delta WHERE aid = :aid;
       215.016  SELECT abalance FROM pgbench_accounts WHERE aid = :aid;
     12512.945  UPDATE pgbench_tellers SET tbalance = tbalance + :delta WHERE tid = :tid;
      2614.245  UPDATE pgbench_branches SET bbalance = bbalance + :delta WHERE bid = :bid;
       214.832  INSERT INTO pgbench_history (tid, bid, aid, delta, mtime) VALUES (:tid, :bid, :aid, :delta, CURRENT_TIMESTAMP);
       216.340  END;

```

