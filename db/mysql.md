# CLI

```sh
# 서버 접속해서 SQL 실행
mysql -uroot -p'password' -h 172.31.1.2 -e "select id from information_schema.processlist where user like 'appuser'"

# slow query 죽이기
for i in $(mysql -uroot -p'password' -h 172.31.1.2 -e  "select id from information_schema.processlist where user like 'appuser' and time > 10"); do
  mysql -uroot -p'password' -h 172.31.1.2 -e "kill ${i}"
done
```



# Import, Export

```sh
# export database
mysqldump testdb > testdb.txt

# export table
mysqldump testdb testtable > testtable.txt

# add drop table
mysqldump -p –user=username –add-drop-table testdb testtable > testtable.txt

# import database
mysql -u username -p < testtable.txt

# import table
mysql -u username -p –database=testdb < testtable.txt
```



# Issues

```sql
# 의도적으로 slow query 만들기
select 'a' where sleep(5)=0;
```

