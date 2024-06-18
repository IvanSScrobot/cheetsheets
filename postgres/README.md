### To connect to a DB:

```
psql -U username -d database
 
#to connect from another node:
psql -U username -h db-hostname -d database

#to log in without a password:
sudo -u user_name psql db_name
#or
sudo -su postgres psql -U user_admin -d database
 
#To reset the password if you have forgotten:
ALTER USER user_name WITH PASSWORD 'new_password';
```

### Useful commands:


```
#To get all tables:
\d+
 
#To get history:
\s
 
#to display databases
\l
 
#connect to new database
\c 
 
#To get all connections:
SELECT * FROM pg_stat_activity WHERE datname = 'dbname';
 
#To kill connections:
SELECT
    pg_terminate_backend(pid)
FROM
    pg_stat_activity
WHERE
    -- don't kill my own connection!
    pid <> pg_backend_pid()
    -- don't kill the connections to other databases
    AND datname = 'database_name'
    ;
 
#to check if DB exists:
select exists(
 SELECT datname FROM pg_catalog.pg_database WHERE lower(datname) = lower('dbname')
);
 
#To list all tables:
#In all schemas:
 \dt *.*
 
#In a particular schema:
\dt schema_name.*
SELECT * FROM information_schema.tables WHERE table_schema = 'schema_name';
 
#list all schemas
SELECT schema_name FROM information_schema.schemata;
 
#To change schema, you can try
 
SET search_path TO schena_name
 
#to check if hot standby serves read-only queries
SELECT pg_is_in_recovery();
```

### To view schema privileges:
```
\dn+
 
#or
 
WITH "names"("name") AS (
  SELECT n.nspname AS "name"
    FROM pg_catalog.pg_namespace n
      WHERE n.nspname !~ '^pg_'
        AND n.nspname <> 'information_schema'
) SELECT "name",
  pg_catalog.has_schema_privilege(current_user, "name", 'CREATE') AS "create",
  pg_catalog.has_schema_privilege(current_user, "name", 'USAGE') AS "usage"
    FROM "names";
```


============
### To check Lag between Master and Secondary nodes:
https://www.postgresql.org/docs/12/functions-admin.html#FUNCTIONS-ADMIN-BACKUP-TABLE

0. Check state of replication
```select * from pg_stat_replication;```

1. On master, to tet the master's current WAL, run
```
SELECT pg_current_wal_lsn();
```
it returns:

 pg_current_wal_lsn 
--------------------
 4/100146B8
(1 row)


2. On secondary, get the current WAL segments received (flushed or applied/replayed):
```
select pg_is_in_recovery(),pg_is_wal_replay_paused(), pg_last_wal_receive_lsn(), pg_last_wal_replay_lsn(), pg_last_xact_replay_timestamp();
```

 pg_is_in_recovery | pg_is_wal_replay_paused | pg_last_wal_receive_lsn | pg_last_wal_replay_lsn | pg_last_xact_replay_timestamp 
-------------------+-------------------------+-------------------------+------------------------+-------------------------------
 t                 | f                       | 4/10014500              | 4/10014500             | 2024-06-11 18:00:50.673736+00



3. To calculate the size of lag, run:
```
select pg_wal_lsn_diff('4/100146B8','4/10014500');
```
The result:

 pg_wal_lsn_diff 
-----------------
             440

If the number is big, translate it into Gb:
```
select round(__number__/pow(1024,3.0),2) missing_lsn_GiB;
```

4. Also on sec, check 
```
SELECT * FROM pg_stat_wal_receiver;
```

### To rebuild secondary:
0. On secondary, stop Postgres: ```systemctl stop postgresql```
1. Get credentials from /data1/ers_db/pgsql/12.9/data_ers/postgresql.conf
2. Remove data from /data1/ers_db/pgsql/12.9/data_ers/
3. On a primary, in pg_hba.conf check that secondary has access to replication:
replication replicator 192.168.10.10/32 md5
or in our case:
host replication ers_rep samenet md5
4. On secondary, run pg_basebackup
```
pg_basebackup -h <IP ADDRESS> -p 5432 -U ers_rep -Xs -P -R -D /data1/ers_db/pgsql/12.9/data_ers/
#in ansible:
shell: PGPASSWORD='{{ ers_db_rep_pwd }}' pg_basebackup -h {{ groups['exportservicedb'][0] }} -p {{ postgres_port }} -D {{ ers_db_pgdata_dir }} -P -X stream -U {{ ers_db_rep_user }}
```

Note that pg_basebackup waits for the checkpoint before it starts the backup. Use  ‘-c fast’ if take too long, or in master, in pgsql, run:
```CHECKPOINT;
```
5. Change permissions: ``` chown -R postgres.postgres /data1/ers_db/pgsql/12.9/data_ers/ ```
6. ```sudo systemctl start postgresql```


### For switchover to secondary:
1. Verify Standby Synchronization
2. Allow secondary IP in master
3. Stop master
4. Promote secondary:
```psql -c "SELECT pg_promote();"```
"To trigger failover of a log-shipping standby server, run pg_ctl promote, call pg_promote, or create a trigger file with the file name and path specified by the promote_trigger_file"

5. Create standby.signal on old master
touch /data1/ers_db/pgsql/12.9/data_ers/standby.signal
6. Update postgresql.auto.conf on old master with the new master details:
primary_conninfo = 'host=db1-ers-{$env}.globalrelay.net port=5432 user=ers_rep password={{ ers_db_rep_pwd }}
7. Start old primary as secondary stand-by
?is it sufficient to just `systemct start postgrsql`?
/usr/lib/postgresql/12/bin/pg_ctl -D /var/lib/postgresql/12/main start

check logs, they should have something like "Starting as a standby"
8. Check that the new secondary is standby and in-sync
```


select * from pg_stat_wal_receiver;
select pg_last_wal_receive_lsn(), pg_last_wal_replay_lsn(), pg_last_xact_replay_timestamp();
```
SELECT pg_is_in_recovery();
<!-- This query will return a single boolean value:  -->
<!-- true if the server is a standby and false if it's the primary. -->

On the new master, insert some data and then check on the secondary
```
create DATABASE test_db;
\c test_db

create TABLE test_table (
    id serial PRIMARY KEY,
    name varchar(100),
    age int
);

INSERT INTO test_table (name, age) VALUES
    ('Alice', 28),
    ('Bob', 32),
    ('Carol', 24);

```    
https://medium.com/@umairhassan27/setting-up-postgresql-replication-on-slave-server-a-step-by-step-guide-1ff36bb9a47f
https://medium.com/@umairhassan27/postgresql-replication-switchover-a-step-by-step-guide-d42107d860
