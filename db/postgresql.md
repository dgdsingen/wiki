# Backup & Restore

> [PostgreSQL DB Backup 및 Restore](https://browndwarf.tistory.com/12) 

```sh
sudo su - postgres
cd /usr/pgsql-12/bin

# check database
psql
\l

# 필요시 password 변경
alter user postgres with password '1212';

# backup: 여기서 password 입력함
pg_dump -d postgres -h localhost -U postgres -F t -v > bak.tar

# restore
pg_restore -d postgres -h localhost -U postgres -F t -v bak.tar
```



# NVL

```sql
SELECT COALESCE(MAX(ID), 0) FROM TEST
```

