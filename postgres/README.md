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