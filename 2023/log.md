```
mkdir -p /log/pg_log
chown -R postgres /log/pg_log
chown -R postgres:postgres /log/pg_log
ALTER SYSTEM SET log_directory = '/log/pg_log';
ALTER SYSTEM SET log_filename = 'postgresql-%u.log';
ALTER SYSTEM SET log_filename = 'postgresql.log';
SELECT pg_reload_conf();
```























