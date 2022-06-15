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

