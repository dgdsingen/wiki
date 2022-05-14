# Oracle 11g XE on Ubuntu

## Ready

- sudo groupadd dba
- sudo vi /etc/group
    - dba에 dgdsingen 추가
- sudo ufw allow 8080, 1521


## XE Install

Installing Oracle 11g R2 Express Edition on Ubuntu 64-bit

The Installer released by Oracle is only meant for 64-bit (x86_64) systems. If you wish to install the 32-bit version , see installing oracle xe on ubuntu 32-bit

It will consume, at most, processing resources equivalent to one CPU.

Only one installation of Oracle Database XE can be performed on a single computer.

The maximum amount of user data in an Oracle Database XE database cannot exceed 11 GB.

The maximum amount of RAM that Oracle XE uses cannot exceed 1 GB, even if more is available.

Now the steps for Installation :

1) Download the Oracle 11gR2 express edition installer from the link given below:

[http://www.oracle.com/technetwork/products/express-edition/downloads/index.html](http://www.oracle.com/technetwork/products/express-edition/downloads/index.html)

( You will need to create a free oracle web account if you don't already have it )

2) Unzip it :

unzip oracle-xe-11.2.0-1.0.x86_64.rpm.zip

3) Install the following packages :

sudo apt-get install alien libaio1 unixodbc vim

4) Convert the red-hat ( rpm ) package to Ubuntu-package :

sudo alien --scripts -d oracle-xe-11.2.0-1.0.x86_64.rpm

(Note: this may take a while , till that time you can go for step 5 )

5) Do the following pre-requisite things:

a) Create a special chkconfig script :

The Red Hat based installer of Oracle XE 11gR2 relies on /sbin/chkconfig, which is not used in Ubuntu. The chkconfig package available for the current version of Ubuntu produces errors and my not be safe to use. Below is a simple trick to get around the problem and install Oracle XE successfully:

sudo vim /sbin/chkconfig


```
#!/bin/bash
# Oracle 11gR2 XE installer chkconfig hack for Ubuntu
file=/etc/init.d/oracle-xe
if [[ ! `tail -n1 $file | grep INIT` ]]; then
echo >> $file
echo '### BEGIN INIT INFO' >> $file
echo '# Provides: OracleXE' >> $file
echo '# Required-Start: $remote_fs $syslog' >> $file
echo '# Required-Stop: $remote_fs $syslog' >> $file
echo '# Default-Start: 2 3 4 5' >> $file
echo '# Default-Stop: 0 1 6' >> $file
echo '# Short-Description: Oracle 11g Express Edition' >> $file
echo '### END INIT INFO' >> $file
fi
update-rc.d oracle-xe defaults 80 01
```


Save the above file and provide appropriate execute privilege :

chmod 755 /sbin/chkconfig

b) Set the Kernel parameters :

Oracle 11gR2 XE requires to set the following additional kernel parameters:

sudo vim /etc/sysctl.d/60-oracle.conf

(Enter the following)


```
# Oracle 11g XE kernel parameters  
fs.file-max=6815744  
net.ipv4.ip_local_port_range=9000 65000  
kernel.sem=250 32000 100 128 
kernel.shmmax=536870912 
```


(Save the file)

Note: kernel.shmmax = max possible value , e.g. size of physical RAM ( in bytes e.g. 512MB RAM == 512*1024*1024 == 536870912 bytes )

Verify the change :

sudo cat /etc/sysctl.d/60-oracle.conf

Load new kernel parameters:

sudo service procps start

Verify: sudo sysctl -q fs.file-max

-> fs.file-max = 6815744

c) Increase the system swap space : Analyze your current swap space by following command :

free -m

Minimum swap space requirement of Oracle 11gR2 XE is 2 GB . In case, your is lesser , you can increase it by following steps in my one of previous post .

d) make some more required changes :

i) ln -s /usr/bin/awk /bin/awk

ii) mkdir /var/lock/subsys

iii) touch /var/lock/subsys/listener

6) Now you are ready to install Oracle 11gR2 XE. Go to the directory where you created the ubuntu package file in Step 4 and enter following commands in terminal :

i) sudo dpkg --install oracle-xe_11.2.0-2_amd64.deb

## Shared memory problem

Trouble-shooting oracle 11g installation on Ubuntu In my one of the previous post ( [http://meandmyubuntulinux.blogspot.com/2012/05/installing-oracle-11g-r2-express.html](http://meandmyubuntulinux.blogspot.com/2012/05/installing-oracle-11g-r2-express.html) ) , I told you about steps for installation of Oracle 11g R2 express edition on Ubuntu. But I found out that many of you are facing problems in installing using the given steps. So, I came up with an idea of adding a trouble-shooter post which can enable you to have a hassle-free installation experience.

Most of the time, if you followed the steps in previous post then you should be able to reach at least up to the configuration part ( Step # 6(ii) ). If you face any problem before this step then you must perform re-installation. For this do the following :

1) Enter the following command on terminal window :


```
sudo -s
/etc/init.d/oracle-xe stop
ps -ef | grep oracle | grep -v grep | awk '{print $2}' | xargs kill
dpkg --purge oracle-xe
rm -r /u01
rm /etc/default/oracle-xe
update-rc.d -f oracle-xe remove
```


2) Follow the steps given in the previous post to install the Oracle 11g XE again.

Now, once you have reached the configuration part. Do the following to avoid getting MEMORY TARGET error ( ORA-00845: MEMORY_TARGET not supported on this system ) :


```
sudo rm -rf /dev/shm
sudo mkdir /dev/shm
sudo mount -t tmpfs shmfs -o size=2048m /dev/shm
```


(here size will be the size of your RAM in MBs ).

The reason of doing all this is that on a Ubuntu system /dev/shm is just a link to /run/shm but Oracle requires to have a seperate /dev/shm mount point.

3) Next you can proceed with the configuration and other consequent steps.

To make the change permanent do the following :

a) create a file named S01shm_load in /etc/rc2.d :

sudo vim /etc/rc2.d/S01shm_load

Now copy and paste following lines into the file :


```
#!/bin/sh
case "$1" in
start) mkdir /var/lock/subsys 2>/dev/null
       touch /var/lock/subsys/listener
       rm /dev/shm 2>/dev/null
       mkdir /dev/shm 2>/dev/null
       mount -t tmpfs shmfs -o size=2048m /dev/shm ;;
*) echo error
   exit 1 ;;
esac
```


b) Save the file and provide execute permissions :

chmod 755 /etc/rc2.d/S01shm_load

This will ensure that every-time you start your system, you get a working Oracle environment.

## Finalize

ii) sudo /etc/init.d/oracle-xe configure

Enter the following configuration information:

A valid HTTP port for the Oracle Application Express (the default is 8080)

A valid port for the Oracle database listener (the default is 1521)

A password for the SYS and SYSTEM administrative user accounts

Confirm password for SYS and SYSTEM administrative user accounts

Whether you want the database to start automatically when the computer starts (next reboot).

7) Before you start using Oracle 11gR2 XE you have to set-up more things :

a) Set-up the environmental variables :

Add following lines to your .bashrc :


```
export ORACLE_HOME=/u01/app/oracle/product/11.2.0/xe
export ORACLE_SID=XE
export NLS_LANG=`$ORACLE_HOME/bin/nls_lang.sh`
export ORACLE_BASE=/u01/app/oracle
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$LD_LIBRARY_PATH
export PATH=$ORACLE_HOME/bin:$PATH
```


b) execute your .profile to load the changes:

. ./.profile

8) Start the Oracle 11gR2 XE :

sudo service oracle-xe start

The output should be similar to following :

user@machine:~$ sudo service oracle-xe start

Starting Oracle Net Listener.

Starting Oracle Database 11g Express Edition instance.

user@machine:~$

8) Create your user :

a) start sqlplus and login as sys :

sqlplus sys as sysdba

( provide the password you gave while configuring the oracle in Step 6 (ii) ).

This should come to following :

SQL*Plus: Release 11.2.0.2.0 Production on Wed May 9 12:12:16 2012

Copyright (c) 1982, 2011, Oracle. All rights reserved.

Enter password:

Connected to:

Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

SQL>

b) Enter following on the sql prompt : Replace username and password by your desired ones.

SQL> create user username identified by password;

User created.

SQL> grant connect,resource to username;

Grant succeeded.

9) Now as you have created the user , you can login to it :

user@machine:~$ sqlplus

SQL*Plus: Release 11.2.0.2.0 Production on Wed May 9 12:28:48 2012

Copyright (c) 1982, 2011, Oracle. All rights reserved.

Enter user-name: temp Enter password:

Connected to:

Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

SQL> select 2+2 from dual;

2+2

---
4

SQL>


## tnsnames.ora


```
# tnsnames.ora Network Configuration File:

XE =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = XE)
    )
  )

EXTPROC_CONNECTION_DATA =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC_FOR_XE))
    )
    (CONNECT_DATA =
      (SID = PLSExtProc)
      (PRESENTATION = RO)
    )
  )
```

# Config

## Oracle RAC + MyBatis XML

LOAD_BALANCE를 ON으로 하면 2대에 LB로 붙고, OFF로 할 경우 Primary에만 붙는다. (Active - Standby)

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="oracle.jdbc.OracleDriver"/>
        <property name="url" value="jdbc:oracle:thin:@(DESCRIPTION =(FAIL_OVER=ON)(LOAD_BALANCE=OFF)(ADDRESS_LIST=(ADDRESS = (PROTOCOL = TCP)(HOST = 10.0.0.1)(PORT = 1527))(ADDRESS = (PROTOCOL = TCP)(HOST = 10.0.0.2)(PORT = 1527)))(CONNECT_DATA =(SERVICE_NAME = TEST)))"/>
        <property name="username" value="ID"/>
        <property name="password" value="PW"/>
      </dataSource>
    </environment>
  </environments>
</configuration>
```



# CLOB

## VARCHAR2 -> CLOB 타입 변경

VARCHAR2 타입 컬럼을 직접 CLOB로 변경할 수는 없다. 새로운 컬럼을 추가해서 데이터 복사 후 기존 컬럼을 삭제 & 재생성 & 데이터 복구해준다.

```sql
ALTER TABLE SOME_TABLE ADD (NEW_COL CLOB);
UPDATE SOME_TABLE SET NEW_COL = OLD_COL;
ALTER TABLE SOME_TABLE DROP COLUMN OLD_COL;
ALTER TABLE SOME_TABLE ADD (OLD_COL CLOB);
UPDATE SOME_TABLE SET OLD_COL = NEW_COL;
ALTER TABLE SOME_TABLE DROP COLUMN NEW_COL;
```



# MERGE INTO

MERGE INTO문을 사용해 INSERT OR UPDATE를 만들 수 있다. (Oracle 9i 이상에서만 사용 가능)

아래 쿼리는 테이블에 kang이라는 id가 존재할 경우엔 flag를 Y로 update하고, 존재하지 않으면 id가 kang이고 flag가 Y인 row를 insert한다.

```sql
MERGE INTO table
USING DUAL
ON (id = ? AND pswd = ?)
WHEN MATCHED THEN
  UPDATE SET pswd = '' WHERE id = ?
WHEN NOT MATCHED THEN
  INSERT VALUES ()
```

# ANSI Join


## CROSS JOIN

크로스 조인은 두 개의 테이블에 대한 Cartesian Product와 같다. 의도적으로 사용하는 것이 아니라면 사용할 일은 없다.


```sql
SELECT first_name, last_name
  FROM emp 
 CROSS JOIN dept;
```

## NATURAL JOIN

내츄럴 조인은 자동적으로 두 테이블에서 같은 이름을 가진 모든 칼럼에 Equi Join을 건다.

조인 칼럼들은 같은 데이터 유형이어야 하며, Alias나 테이블 명과 같은 접두사를 붙일 수 없다. 또 SELECT * 문법을 사용한다면, 공통 칼럼들은 결과 집합에서 한 개만 표현된다.


```sql
 SELECT *
   FROM EMP 
NATURAL JOIN DEPT;
```

## USING 조건

NATURAL JOIN은 같은 이름의 컬럼들이 자동으로 조인이 되지만, USING절을 이용하면 원하는 칼럼만 선택적으로 Equi Join을 할 수 있다.

만일 여러 개의 칼럼이 이름은 같지만 데이터 형이 모두 일치하지 않거나, 몇 개의 칼럼만 선택적으로 조인 조건에 사용하고자 할 때는 Using 절을 이용하여 Equi Join에 사용될 칼럼들을 지정할 수 있다.

USING 절을 이용한 Equi Join에서도 NATURAL JOIN과 마찬가지로, JOIN COLUMN에 대해서는 Alias나 테이블 명과 같은 접두사를 붙일 수 없으며, Natural과 Using의 두 키워드는 상호 배타적으로 사용된다.


```sql
SELECT E.EMPNO, E.DEPTNO, D.DNAME
  FROM EMP E 
  JOIN DEPT D
 USING (DEPTNO);
```

## ON 조건

조인 서술부(ON 절)와 비조인 서술부(WHERE 절)를 분리하여 이해가 쉽다. ON 절을 이용하면 JOIN 이후에 논리 연산과 서브쿼리와 같은 추가 서술을 할 수 있다.

Natural 조인과 달리 ON 조인은 임의의 조인 조건을 지정하거나, 이름이 다른 칼럼끼리 조인 조건으로 사용하거나, 조인할 칼럼을 명시하기 위해서 사용한다.


```sql
-- 여러 테이블의 조인
SELECT E.EMPNO, D.DNAME, B.SAL
  FROM EMP E 
  JOIN DEPT D  
    ON ( E.DEPTNO = D.DEPTNO )
  JOIN BONUS B
    ON ( E.ENAME = B.ENAME );
 
-- WHERE 절과의 혼용
SELECT E.ENAME, E.DEPTNO, D.DEPTNO, D.DNAME 
  FROM EMP E 
  JOIN DEPT D
    ON ( E.DEPTNO = D.DEPTNO )
 WHERE E.EMPNO >= 7000;
 
-- ON 절의 조건 추가
SELECT E.ENAME, E.MGR, D.DEPTNO, D.DNAME
  FROM EMP E 
  JOIN DEPT D 
    ON ( E.DEPTNO = D.DEPTNO  AND  E.MGR = 7698 );
 
-- EXIST 절 사용
SELECT E.EMPNO, E.ENAME, D.DEPTNO, D.DNAME, B.BONUS
  FROM EMP E 
  JOIN DEPT D 
    ON ( E.DEPTNO = D.DEPTNO 
         AND NOT EXISTS (SELECT 1 
                           FROM BONUS B 
                          WHERE E.ENAME = B.ENAME) );
```

## INNER JOIN

INNER JOIN시 INNER 구문은 생략 가능하다. (JOIN 구문만 사용시 디폴트는 INNER JOIN)


```sql
-- 아래 두 쿼리는 동일한 구문이다. 
SELECT E.EMPNO, E.DEPTNO, D.DNAME
  FROM EMP E 
  INNER JOIN DEPT D
    ON E.DEPTNO = D.DEPTNO
 
SELECT E.EMPNO, E.DEPTNO, D.DNAME
  FROM EMP E 
  JOIN DEPT D
    ON E.DEPTNO = D.DEPTNO    
```

## OUTER JOIN


### LEFT OUTER JOIN

Table A와 B가 있을 때 왼쪽에 있는 Table A가 기준이 된다. A와 B를 비교해서같은 것이 있을 때 그 해당 Data를 가져오고, B가 없는 경우에도 가져오는데 B에서 NULL을 가져온다.


```sql
SELECT E.ENAME, E.DEPTNO, D.DNAME 
  FROM EMP 
  LEFT OUTER JOIN DEPT 
    ON (EMP.DEPTNO = DEPT.DEPTNO);
```

### RIGHT OUTER JOIN

Table A와 B가 있을 때 오른쪽에 있는 Table B가 기준이 된다. A와 B를 비교해서 같은 것이 있을 때 그 해당 Data를 가져오고, A가 없는 경우에도 가져오는데 A에서 NULL을 가져온다.


```sql
SELECT E.ENAME, E.DEPTNO, D.DNAME
  FROM EMP 
 RIGHT OUTER JOIN DEPT 
    ON (EMP.DEPTNO = DEPT.DEPTNO);
```

### FULL OUTER JOIN

Table A와 B가 있을 때 (Table A, B 모두 기준), Left Outer Join과 Right Outer Join의 결과를 UNION으로 합친 것과 같다.


```sql
SELECT E.ENAME, E.DEPTNO, D.DNAME
  FROM EMP 
  FULL OUTER JOIN DEPT 
    ON (EMP.DEPTNO = DEPT.DEPTNO);  
```

# Job

## Job 등록 예제


```sql
-- 권한 부여 (APRECRUITDBA에서 실행)
GRANT EXECUTE ON P_CONSTANTAPPLICATIONJOB_BATCH TO TEST;
GRANT EXECUTE ON P_DELRESUMETEMPSAVEJOB_BATCH TO TEST;
GRANT EXECUTE ON P_EXIFJOB_BATCH TO TEST;
GRANT EXECUTE ON P_RETAINJOB_BATCH TO TEST;
 
-- 배치 RUN 프로시저 등록 (TEST에서 실행)
CREATE OR REPLACE PROCEDURE P_CONSTANTAPPLICATIONJOB_RUN
IS
    AV_TABLEID VARCHAR2(4000);
    AV_ERROR_NUM VARCHAR2(4000);
    AV_ERROR_MSG VARCHAR2(4000);
BEGIN
    APRECRUITDBA.P_CONSTANTAPPLICATIONJOB_BATCH(TO_CHAR(SYSDATE, 'YYYYMMDD'), AV_TABLEID, AV_ERROR_NUM, AV_ERROR_MSG);
    DBMS_OUTPUT.PUT_LINE(AV_TABLEID);
    DBMS_OUTPUT.PUT_LINE(AV_ERROR_NUM);
    DBMS_OUTPUT.PUT_LINE(AV_ERROR_MSG);
END;
 
-- Job 등록 (TEST에서 실행)
DECLARE
  X NUMBER;
BEGIN
  SYS.DBMS_JOB.SUBMIT
    ( job       => X 
     ,what      => 'P_CONSTANTAPPLICATIONJOB_RUN;'
     ,next_date => TRUNC(SYSDATE) + 1 + 12/24
     ,INTERVAL  => 'TRUNC(SYSDATE) + 1 + 12/24' -- 매일 12시마다
     --,INTERVAL  => 'TRUNC(SYSDATE) + 1 + 0/24' -- 매일 0시마다
     --,INTERVAL  => 'SYSDATE + 5/24/60' -- 매 5분마다
     ,no_parse  => TRUE
    );
  SYS.DBMS_OUTPUT.PUT_LINE('Job Number is: ' || to_char(x)); -- 이부분은 job큐의 번호가 됩니다.
END;
 
-- Job 확인
SELECT * 
FROM user_jobs
WHERE what LIKE '%_RUN%'
 
-- 배치 히스토리 확인
SELECT TO_CHAR(regDate, 'yy/mm/dd hh24:mi:ss'), a.*
  FROM APRECRUITDBA.H_SP001MT a
ORDER BY logdate DESC
```

## Interval

Job이 실행될때 구해지는 interval 값이 job 실행시점보다 과거라면 failure로 빠진다.

예를 들어 job이 2015-11-24 13:00:00에 돌았는데 interval이 TRUNC(SYSDATE) + 12/24라면 다음 실행 시점은 2015-11-24 12:00:00가 된다. 그러면 다음 실행시점이 현재 실행시점보다 과거이므로 failure 처리가 되버린다. 

그러므로 interval은 다음과 같이 처리하자

- '매일 정해진 시각'에 한번 도는 것이라면 -> TRUNC(SYSDATE) + 1 + 12/24 (매일 12시)

- '일정 간격'으로 한번씩 도는 것이라면 -> SYSDATE + 12/24 (매 12시간)

# Trigger

## Exceptions

Exceptions raised by triggers have a statement severity; they roll back the statement that caused the trigger to fire.

This rule applies to nested triggers (triggers that are fired by other triggers). If a trigger action raises an exception (and it is not caught), the transaction on the current connection is rolled back to the point before the triggering event. For example, suppose Trigger A causes Trigger B to fire. If Trigger B throws an exception, the current connection is rolled back to the point before the statement in Trigger A that caused Trigger B to fire. Trigger A is then free to catch the exception thrown by Trigger B and continue with its work. If Trigger A does not throw an exception, the statement that caused Trigger A, as well as any work done in Trigger A, continues until the transaction in the current connection is either committed or rolled back. However, if Trigger A does not catch the exception from Trigger B, it is as if Trigger A had thrown the exception. In that case, the statement that caused Trigger A to fire is rolled back, along with any work done by both of the triggers.

You might want a trigger action to be able to abort the triggering statement or even the entire transaction.

- [Aborting statements and transactions](https://docs.oracle.com/javadb/10.8.3.0/devguide/cdevspecial93497.html)

Parent topic

- [Programming trigger actions](https://docs.oracle.com/javadb/10.8.3.0/devguide/cdevspecial27163.html)

Related concepts

- [Trigger action overview](https://docs.oracle.com/javadb/10.8.3.0/devguide/cdevspecial53165.html)
- [Performing referential actions](https://docs.oracle.com/javadb/10.8.3.0/devguide/cdevspecial76763.html)
- [Accessing before and after rows](https://docs.oracle.com/javadb/10.8.3.0/devguide/cdevspecial67260.html)

Related reference

- [Examples of trigger actions](https://docs.oracle.com/javadb/10.8.3.0/devguide/cdevspecial13670.html)

# Tuning

내가 짠 쿼리의 plan을 보자. cost가 나온다. 또 option이 full인지, index를 타는지 알 수 있다. cost가 지나치게 높다면 뭔가 잘못된 것이다. 

대량 데이터 처리 시 트리거에 걸리면 미친듯이 느려진다.

## JOIN & NOT IN

```sql
/* 이전 쿼리 */ 
SELECT * FROM (SELECT *  FROM tbl  ORDER BY LIKECNT DESC)  WHERE rownum < 4  --Top 3개 추출
UNION ALL
SELECT * FROM (SELECT   * -- Top3개를 제외한 데이터를 최신 순으로 조회
                 FROM   tbl
                WHERE   seq NOT IN  (SELECT seq FROM (SELECT seq FROM tbl ORDER BY LIKECNT DESC )  WHERE rownum < 4 ) 
               ORDER BY regdate DESC)
-- 데이터 1만개 있을 시에 38초 소요됨. NOT IN 조건의 대상이 고정 값이 아닌 테이블 조회방식(실시간 변동적)이고 그 테이블이 tbl이므로 성능 문제 발생 

/* 변경된 쿼리 */ 
SELECT t2.*, CASE WHEN rk <=3 THEN rk ELSE 99 END rk1  --rk(추천수)가 3위 안에 들면 그 순위(1,2,3)를 사용하고 그 이후는 모두 99로 동점처리
  FROM (SELECT t1.*, rank() OVER (ORDER BY likecnt DESC, rownum DESC) rk, rank () OVER (ORDER BY regdate DESC) rk2 
	      FROM tbl t1) t2
ORDER BY rk1 ASC, rk2 ASC  --rk정렬 조건에서 1,2,3등까지만 우선순위가 생기고 나머지는 모두 99이므로 두 번째 정렬조건인 rk2(최신순)로 정렬됨
-- 데이터 1만개 있을 시에 0.28초에 조회됨
```

## WHERE

아래 중 어느 것이 cost가 높을까?

```sql
-- 1번 SQL
SELECT   *
  FROM	 TEST
 WHERE	 a = '1'
  AND	 b = RPAD('123', 18, ' ');
```

```sql
-- 2번 SQL
SELECT   *
  FROM	 TEST
 WHERE	 a = '1'
  AND	 TRIM(b) = '123';
```

전자는 cost가 3정도, 후자는 cost가 8000이 넘게 나온다. 왜 이럴까? 조건을 비교하는 테이블의 컬럼 (좌측)에 TRIM 함수를 사용했으니 index를 타지 못하고 full로 돈다. 그래서.. 만약 다음과 같은 쿼리를 사용하고 싶다면

```sql
SELECT  *
  FROM  TEST
 WHERE  a || b = '1123';
```

다음과 같이 바꾸면 인덱스를 타서 훨씬 빨라진다.

```sql
SELECT  * 
  FROM  bd01vin
 WHERE  (a, b) IN (
('1', '123'), 
('1', '234')
);
```

## IN vs Exists

일반적으로 EXISTS절이 더 우수한 성능을 보인다. EXISTS는 해당 DATA를 갖고 있는 ROW가 존재하는지만 체크하고 끝나고 IN은 실제 존재하는 데이터의 값을 모두 체크한다.

JOIN되는 COLUMN에 NULL을 갖는 ROW가 존재할 시에 NOT EXISTS 는 TRUE, NOT IN은 FALSE를 출력한다. NOT IN을 사용하면 조건에 맞는 데이터가 있어도 NULL이 있으면 NOT ROWS SELECTED가 나오게 되므로 NVL로 NULL 처리를 해야한다. 

```sql
SELECT COLUMNS
  FROM TABLES
 WHERE EXISTS( CHECK값 );
```

# Hint

## BYPASS_UJVC

 Oracle 11g부터 지원되지 않으므로, Updatable Join View가 필요할 경우 MERGE 문으로 처리하자.


### Undocumented HIT (잘못되어도 오라클에서 보장 하지 않음)

일반적으로 특정 테이블을 Update하기 위해서는 WHERE절에 EXISTS 또는 IN 등의 Sub-Query로 조인 조건을 만족하는 Row를 먼저 Check하고, 조건을 만족하는 Row에 대하여 SET 절에서 필요한 데이터를 검색하여 Update하는 것이 보통이다.

이 때, Update 해야 하는 Row가 많을 경우 WHERE절이나 SET절에서 테이블을 반복적으로 Random 액세스해야 하는 부담이 발생하므로 처리 범위가 넓은 Update문의 경우에는 'Updatable Join View'를 활용할 필요가 있다.

이 때, 조인되는 2개의 테이블은 반드시 1:1 또는 1:M의 관계여야 하며, Update되는 컬럼의 테이블은 M쪽 집합이어야 한다.

이것은 1쪽 집합인 Parent table의 조인 컬럼이 UK 또는 PK로 설정되어 있어야 함을 의미한다.

이 조건을 만족하지 못하는 Updatable Join View는 에러를 Return하며 실행되지 않는다. (ORA-01779 cannot modify a column which maps to a non key-preserved table)

그러나, 일반적으로 View 또는 2개 이상의 테이블을 조인한 집합을 엑세스하는 경우가 많으므로 위의 UK나 PK Constraint를 설정하기 어려운 것이 현실이다.

따라서, 이러한 Constraint를 피해서 Updatable Join View를 사용할 수 있도록 BYPASS_UJVC 라는 힌트를 사용하여 튜닝할 수 있다.


### Example


#### TEST 1 환경

Constraint :

- DETP.DEPTNO CONSTRAINT PK_DEPT PRIMARY KEY
- EMP.EMPNO CONSTRAINT PK_EMP PRIMARY KEY
- EMP.DEPTNO CONSTRAINT FK_DEPTNO REFERENCES DEPT


#### VIEW 생성


```
CREATE OR REPLACE VIEW empdept_v
AS
SELECT x.empno, x.ename, x.job, y.dname, y.deptno
FROM   emp x, dept y
WHERE x.deptno = y.deptno;
 
VIEW created.
```

#### UPDATABLE JOIN VIEW 내용


```
SQL> SELECT * FROM empdept_v;
 
     EMPNO ENAME      JOB       DNAME              DEPTNO
---------- ---------- --------- -------------- ----------
      7369 SMITH      CLERK     RESEARCH               20
      7499 ALLEN      SALESMAN SALES                  30
      7521 WARD       SALESMAN SALES                  30
      7566 JONES      MANAGER   RESEARCH               20
      7654 MARTIN     SALESMAN SALES                  30
      7698 BLAKE      MANAGER   SALES                  30
      7782 CLARK      MANAGER   ACCOUNTING             10
      7788 SCOTT      ANALYST   RESEARCH               20
      7839 KING       PRESIDENT ACCOUNTING             10
      7844 TURNER     SALESMAN SALES                  30
      7876 ADAMS      CLERK     RESEARCH               20
      7900 JAMES      CLERK     SALES                  30
      7902 FORD       ANALYST   RESEARCH               20
      7934 MILLER     CLERK     ACCOUNTING             10
 
14 ROWS selected.
```

#### 1쪽 컬럼 갱신


```
UPDATE empdept_v
   SET dname = 'AP_TUNNING'
 WHERE empno = '7369';
 
ERROR at line 2:
ORA-01779: cannot MODIFY a COLUMN which maps TO a non key-preserved TABLE
```

#### 1쪽 컬럼 갱신 WITH bypass_ujvc HIT


```
UPDATE /*+ bypass_ujvc */
       empdept_v
   SET dname = 'AP_TUNNING'
 WHERE empno = '7369';
 
1 ROW updated.
```

#### 변경 내역 확인


```
SQL> SELECT * FROM dept;
 
    DEPTNO DNAME          LOC
---------- -------------- -------------
        10 ACCOUNTING     NEW YORK
        20 AP_TUNNING     DALLAS
        30 SALES          CHICAGO
        40 OPERATIONS     BOSTON
 
SQL> SELECT * FROM empdept_v;
 
     EMPNO ENAME      JOB       DNAME              DEPTNO
---------- ---------- --------- -------------- ----------
      7369 SMITH      CLERK     AP_TUNNING             20
      7499 ALLEN      SALESMAN SALES                  30
      7521 WARD       SALESMAN SALES                  30
      7566 JONES      MANAGER   AP_TUNNING             20
      7654 MARTIN     SALESMAN SALES                  30
      7698 BLAKE      MANAGER   SALES                  30
      7782 CLARK      MANAGER   ACCOUNTING             10
      7788 SCOTT      ANALYST   AP_TUNNING             20
      7839 KING       PRESIDENT ACCOUNTING             10
      7844 TURNER     SALESMAN SALES                  30
      7876 ADAMS      CLERK     AP_TUNNING             20
      7900 JAMES      CLERK     SALES                  30
      7902 FORD       ANALYST   AP_TUNNING             20
      7934 MILLER     CLERK     ACCOUNTING             10
 
14 ROWS selected.
```

#### TEST 2 환경


```
 제약조건 : Constraint 없음.
```

#### 의미상 M쪽 집합을 갱신함


```
UPDATE
(SELECT b.dname, a.ename
   FROM emp a,
        dept b
 WHERE a.deptno = b.deptno
    AND a.empno = '7369')
SET ename = 'KIMDOL';
 
ERROR at line 5:
ORA-01779: cannot MODIFY a COLUMN which maps TO a non key-preserved TABLE
```


Constraint가 없으므로 oracle은 M쪽 집합인지 알지 못한다.


#### 의미상 M쪽 집합을 갱신, With bypass_ujvc HIT


```
UPDATE /*+ bypass_ujvc */
(SELECT b.dname, a.ename
   FROM emp a,
        dept b
 WHERE a.deptno = b.deptno
      AND a.empno = '7369')
SET dname = 'KIMDOL';
 
1 ROW updated.
```

# Issues

## Lock 확인 및 해제

```sql
SELECT vo.session_id,do.object_name, do.owner, do.object_type,do.owner, 
       vo.xidusn, vo.locked_mode 
FROM v$locked_object vo , dba_objects do 
WHERE vo.object_id = do.object_id;

SELECT COUNT(*)
FROM v$locked_object vo , dba_objects do 
WHERE vo.object_id = do.object_id;

SELECT a.sid, a.serial#
FROM v$session a, v$lock b, dba_objects c
WHERE a.sid=b.sid AND
b.id1=c.object_id AND
b.type='TM' ;

ALTER system KILL SESSION '37,40161';
COMMIT;
```

```sql
/********************************************************************
    SIMPLE SCRIPT TO CHECK LOCKS ON THE SYSTEM
    -- 현재 LOCK 상태를 조회한다.
********************************************************************/

SELECT s.username,
s.sid,
l.type,
l.id1,
l.id2,
l.lmode,
l.request,
p.spid PID
FROM v$lock l,
v$session s,
v$process p
WHERE s.sid = l.sid
  AND p.addr = s.paddr
  AND s.username IS NOT NULL
ORDER BY id1, s.sid, request;
```

```sql
/*************************************************************************************
현재 LOCK 상태와 세션 문자열을 출력한다.
다음 SQL을 실행하여 Lock Requested 가 있다면 현재 wait 상태이다.
wait상태를 풀기 위해 현재 LOCK을 HELD하고 있는 SESSION을 찾아 KILL해야 한다.
찾는 방법은 다음과 같다.
세션별로 Lock Requested가 있는 쿼리의 문장과 Lock Requested가 없는 문장을 비교하여 
쿼리문의 access object가 같다면 Lock Requested가 없는 문장의 세션을 kill 하면 된다.
세션 KILL 구문 ( ALTER SYSTEM KILL SESSION 'Session ID,Serial_NO')을 하면 된다.

참고로 $ORACLE_HOME/rdbms/admin/utllockt.sql 이라는 script가 있다.
이 script는 현재 lock에 대해 tree형식으로 출력한다.
이 script를 사용하면 쉽게 어떤 세션이 어떤 세션의 lock을 wait하는지 확인이 된다.
*************************************************************************************/

SET linesize 132 pagesize 66

--break on kill on username on terminal
COLUMN KILL heading 'Kill String' format a13
COLUMN res heading 'Resource Type' format 999
COLUMN id1 format 9999990
COLUMN id2 format 9999990
COLUMN lmode heading 'Lock Held' format a20
COLUMN request heading 'Lock Requested' format a20
COLUMN serial# format 99999
COLUMN username format a10 heading "Username"
COLUMN terminal heading Term format a6
COLUMN tab format a15 heading "Table Name"
COLUMN owner format a9
COLUMN Address format a18

SELECT NVL(S.USERNAME,'Internal') username,
NVL(S.TERMINAL,'None') terminal,
L.SID||','||S.SERIAL# KILL,
U1.NAME||'.'||SUBSTR(T1.NAME,1,20) tab,
DECODE(L.LMODE, 1, 'No Lock',
2, 'Row Share',
3, 'Row Exclusive',
4, 'Share',
5, 'Share Row Exclusive',
6, 'Exclusive',NULL) lmode,
DECODE(L.REQUEST, 1, 'No Lock',
  2, 'Row Share',
  3, 'Row Exclusive',
  4, 'Share',
  5, 'Share Row Exclusive',
  6, 'Exclusive',NULL) request,
SQL.SQL_TEXT SQL_TEXT
FROM V$LOCK L,
V$SESSION S,
V$SQLTEXT SQL,
SYS.USER$ U1,
SYS.OBJ$ T1
WHERE L.SID = S.SID
  AND S.SQL_ADDRESS = SQL.ADDRESS
  AND S.SQL_HASH_VALUE = SQL.HASH_VALUE
  AND T1.OBJ# = DECODE(L.ID2,0,L.ID1,L.ID2)
  AND U1.USER# = T1.OWNER#
  AND S.TYPE != 'BACKGROUND'
ORDER BY 1,2,5;
```

```sql
-- 해당 세션의 SQL statement를 확인한다.
SELECT sqltext
FROM v$sqltext a, v$session b
WHERE a.address = b.sql_address
  AND a.hash_value = b.sql_hash_value
  AND b.sid = [세션 ID]
ORDER BY peice;
```

```sql
-- 테이블에 락걸린 정보 확인

SELECT B.TYPE, C.OBJECT_NAME, A.SID, A.SERIAL#    
FROM  V$SESSION A, V$LOCK B, DBA_OBJECTS C    
WHERE A.SID=B.SID
    AND B.ID1=C.OBJECT_ID
    AND B.TYPE='TM'
    AND C.OBJECT_NAME='테이블 명';
```

 ```sql
-- 확인 된 정보에서 SID, SERIAL# 번호를 통해서 락을 KILL 한다.
-- ex ) ALTER SYSTEM KILL SESSION '58, 6015';
ALTER SYSTEM KILL SESSION 'SID, SERIAL#'
 ```

```sql
-- Lock 정보 확인
SELECT 
    A.SESSION_ID AS SESSION_ID,
    B.SERIAL# AS SERIAL_NO,
    A.OS_USER_NAME AS OS_USER_NAME,
    A.ORACLE_USERNAME AS ORACLE_USERNAME,
    B.STATUS AS STATUS
FROM V$LOCKED_OBJECT A, V$SESSION B
WHERE A.SESSION_ID = B.SID;
```

```sql
-- Lock 검사

SELECT A.SID,
    DECODE(A.TYPE,
        'MR', 'Media Recovery',
        'RT', 'Redo Thread',
        'UN', 'User Name',
        'TX', 'Transaction',
        'TM', 'DML',
        'UL', 'PL/SQL User Lock',
        'DX', 'Distributed Xaction',
        'CF', 'Control File',
        'IS', 'Instance State',
        'FS', 'File Set',
        'IR', 'Instance Recovery',
        'ST', 'Disk Space Transaction',
        'IR', 'Instance Recovery',
        'ST', 'Disk Space Transaction',
        'TS', 'Temp Segment',
        'IV', 'Library Cache Invalidation',
        'LS', 'Log Start or Switch',
        'RW', 'Row Wait',
        'SQ', 'Sequence Number',
        'TE', 'Extend Table',
        'TT', 'Temp Table',
        A.TYPE) LOCK_TYPE,
    DECODE(A.LMODE,
        0, 'None', /* MON LOCK EQUIVALENT */
        1, 'Null', /* N */
        2, 'Row-S (SS)', /* L */
        3, 'Row-X (SX)', /* R */
        3, 'Row-X (SX)', /* R */
        4, 'Share', /* S */
        5, 'S/Row-X (SSX)', /* C */
        6, 'Exclusive', /* X */
    TO_CHAR(A.LMODE)) MODE_HELD,
    DECODE(A.REQUEST,
        0, 'None', /* MON LOCK EQUIVALENT */
        1, 'Null', /* N */
        2, 'Row-S (SS)', /* L */
        3, 'Row-X (SX)', /* R */
        4, 'Share', /* S */
        5, 'S/Row-X (SSX)', /* C */
        6, 'Exclusive', /* X */
    TO_CHAR(A.REQUEST)) MODE_REQUESTED,
    TO_CHAR(A.ID1) LOCK_ID1,
    TO_CHAR(A.ID2) LOCK_ID2
FROM V$LOCK A
WHERE (ID1,ID2) IN
    (SELECT B.ID1, B.ID2 FROM V$LOCK B
     WHERE B.ID1=A.ID1
        AND B.ID2=A.ID2 AND B.REQUEST>0)
```

```sql
-- Lock 계정 확인
SELECT username, account_status FROM dba_users;
```

```sql
-- Lock 계정 해제
ALTER 계정 account UNLOCK;
```

## Session 정리

```sql
-- Session에 걸린 Query 보기
SELECT a.sid,            -- SID 
       a.status,        -- 상태정보 
       a.process,      -- 프로세스정보 
       a.osuser,        -- 접속자의 OS 사용자 정보 
       b.sql_text,      -- sql 
       c.program      -- 접속 프로그램 
FROM v$session a, 
     v$sqlarea b, 
     v$process c 
WHERE a.sql_hash_value=b.hash_value 
AND a.sql_address=b.address 
AND a.paddr=c.addr 
AND a.status='ACTIVE';  -- 현재 상태가 ACTIVE인것
```

```sql
-- DB 접속 계정별 WAS DB Connection Count (Sub)
select username, machine, count(1) from v$session where username = 'user1' group by username, machine;
select username, machine, count(1) from v$session where username = 'user2' group by username, machine order by count(1);
select username, machine, count(1) from v$session where username = 'user3' group by username, machine order by count(1);
```

```sql
-- DB 접속 계정별 WAS DB Connection Count (Total)
select username, count(1) from v$session group by username order by count(1) desc;
select username, count(1) from v$session where machine like 'was%' group by username order by count(1) desc;
select username, machine, count(1) from v$session group by username, machine order by count(1) desc;
```

```sql
-- Top Query
select * from v$sql where parsing_schema_name = 'user1' order by cpu_time desc, elapsed_time desc;
select * from v$sql where parsing_schema_name = 'user1';
```

## 한글 설정

한글을 지원하는 캐릭터 셋

- KO16KSC5601
- KO16MSWIN949
- UTF8
- AL32UTF8

### 오라클 서버 한글 설정이 안된 경우

현재 서버의 파라미터 설정값을 보려면 다음을 입력한다.

```sql
select * from nls_database_parameters;
```

여기에는 언어, 문자, 시간형식등 현재 지정된 파라미터값들이 전부 나온다. 필요한 정보만 얻어야하므로 다음의 쿼리로 서버의 현재 문자설정값을 뽑아온다.

```sql
select
(select value from nls_database_parameters where parameter = 'NLS_LANGUAGE') || '_' ||
(select value from nls_database_parameters where parameter = 'NLS_TERRITORY') || '.' ||
(select value from nls_database_parameters where parameter = 'NLS_CHARACTERSET') as yourCharterset
from dual;
```

위 쿼리로 얻은 값에 예를들어 AMERICAN_AMERICA.KO16MSWIN949 이런 식으로 KO16MSWIN949 또는 KO16KSC5601 같은 한글 문자셋이 설정이 되어 있어야한다. 한글 문자셋이 없이 다른 값 (예를들어 (AMERICAN_AMERICA.AL32UTF8) 같은.....)이라면 서버에서 한글을 인식 하지 못하는 경우이고 이때 한글로 insert 한 값은 다깨져서 다시 입력해야한다. 아예 한글을 인식할수 없는 코드로 들어가버렸기 때문임. 서버에서 한글문자셋 지정이 안되있는경우에는 sys 권한으로 접속하여 한글 문자셋으로 변경을 먼저 해줘야한다. 그리고 데이터를 insert 해야한다

```sql
SQL> SHUTDOWN IMMEDIATE;
```

만일의 사태를 대비해 DB 백업

```sql
SQL> STARTUP MOUNT;
SQL> ALTER SYSTEM ENABLE RESTRICTED SESSION;
SQL> ALTER SYSTEM SET JOB_QUEUE_PROCESSES=0;
SQL> ALTER SYSTEM SET AQ_TM_PROCESSES=0;
SQL> ALTER DATABASE OPEN;
SQL> ALTER DATABASE CHARACTER SET KO16MSWIN949;
```

주의!!!! 만약 ALTER DATABASE CHARACTER SET KO16MSWIN949;에서 에러날 때 : ALTER DATABASE CHARACTER SET INTERNAL_USE KO16KSC5601 로 일단 변경후 다시 ALTER DATABASE CHARACTER SET KO16MSWIN949 로 변경하니 된다.

## 한글 byte

오라클의 문자 집합에 따라 한글을 인식하는 Byte 길이가 달라진다.

- KO16KSC5601(한글 완성형), KO16MSWIN949 : 한글 한 글자를 2 Byte로 인식
- UTF8/AL32UTF8 : 한글 정렬(order by)이 가능하지만, 한글 한 글자를 3 Byte로 인식

```sql
-- CharacterSet 확인
SELECT * FROM NLS_DATABASE_PARAMETERS WHERE PARAMETER LIKE '%CHARACTERSET%';
```

## 중복 데이터 찾기

다음 쿼리는 데이터를 어떤 처리를 했을 때 (CASE문) 그 처리 결과에서 데이터 중복이 난 경우 그 결과를 가져온다.


```sql
SELECT part_num
FROM
  (SELECT
    CASE
      WHEN BDWORK.F_GET_ALPHA_NUM(SUBSTR(TRIM(ptno),1,18)) = 'N'
      THEN LPAD(SUBSTR(TRIM(ptno),1,18),18,'0')
      ELSE SUBSTR(TRIM(ptno),1,18)
    END AS part_num
  FROM bd01part
  )
GROUP BY part_num
HAVING COUNT(*) > 1;
```


만약 아래와 같은 쿼리를 실행한다면 결과는 나오지 않는다.

왜냐하면 데이터를 처리하기 전에는 중복이 없지만 처리한 후에는 중복이 생기기 때문이다.


```sql
SELECT part_num
FROM
  (SELECT part_num
   FROM bd01part)
GROUP BY part_num
HAVING COUNT(*) > 1;
```


다음은 이 예제를 따온 전체 소스다. 중복 문제와 집합 문제가 섞여 있다.


```sql
SELECT *
FROM
  -- update 친것 중 result에 포함되지 않은 파트 : 36개 --
  (
  SELECT
    CASE
      WHEN BDWORK.F_GET_ALPHA_NUM(SUBSTR(TRIM(ptno),1,18)) = 'N'
      THEN LPAD(SUBSTR(TRIM(ptno),1,18),18,'0')
      ELSE SUBSTR(TRIM(ptno),1,18)
    END AS part_num
  FROM bd01pprc
  WHERE dist_cd = 'EDM'
  AND reg_dt    = '20121211'
  MINUS
  SELECT
    CASE
      WHEN BDWORK.F_GET_ALPHA_NUM(SUBSTR(TRIM(part_num),1,18)) = 'N'
      THEN LPAD(SUBSTR(TRIM(part_num),1,18),18,'0')
      ELSE SUBSTR(TRIM(part_num),1,18)
    END AS part_num
  FROM gwm_ii051_00
  WHERE file_name = 'DELTA1'
  AND bu_cd       = 'KDK0'
  )
WHERE part_num NOT IN
  -- 마스터 중복 파트 34개 --
  (
  SELECT part_num FROM
    (SELECT
      CASE
        WHEN BDWORK.F_GET_ALPHA_NUM(SUBSTR(TRIM(ptno),1,18)) = 'N'
        THEN LPAD(SUBSTR(TRIM(ptno),1,18),18,'0')
        ELSE SUBSTR(TRIM(ptno),1,18)
      END AS part_num
    FROM bd01part
    )
  GROUP BY part_num
  HAVING COUNT(*) > 1
  ) ;
```

## 위도 & 경도로 거리 계산하기


```sql
SELECT 6371*ACOS(COS(TO_NUMBER(LAT1)*0.017453293)*COS(LAT2*0.017453293)*COS((LNG2*0.017453293)-
(TO_NUMBER(LNG1)*0.017453293))+SIN(TO_NUMBER(LAT1)*0.017453293)*SIN(LAT2*0.017453293))
FROM DUAL
```

## LOOP 돌며 테스트 데이터 INSERT


```sql
DECLARE
te NUMBER;
 
BEGIN
    SELECT 0
    INTO te
    FROM DUAL;
 
    WHILE te < 10 LOOP
        INSERT INTO TEST_TABLE VALUES('test' || te, SYSDATE);
 
        SELECT te+1
        INTO te
        FROM DUAL;
      COMMIT;
    END LOOP;
END;
```

## 문자열 안에 작은따옴표(') 사용


```sql
-- 다음과 같이 '' 두개 연속 찍어주면 된다
SELECT ' example : ''yummy'' ' FROM DUAL
```

## table meta data 검색


```sql
SELECT   * 
  FROM   all_tab_comments 
 WHERE   TABLE_NAME LIKE '%' 
   AND   comments   LIKE '%'
```

## source meta data 검색


```sql
SELECT *
  FROM all_source 
 WHERE UPPER(text) LIKE '%text%'
```

## Paging for Oracle


```sql
-- SQL
SELECT   *
  FROM   (SELECT   a.*, ROWNUM rn
		 FROM   a
		WHERE   ROWNUM <= #endNo#)
 WHERE   rn >= #startNo#
 
-- Java
IF (pageNo <= 0)
    pageNo = 1;
startNo = (pageNo * pageSize) - (pageSize - 1);
endNo = startNo + (pageSize - 1);
```

## IN & EXISTS


```sql
-- IN
SELECT   *
  FROM   a
 WHERE   a.id IN (SELECT   b.id
		    FROM   b
		   WHERE   b.condition = 'temp')
 
-- EXISTS
SELECT   *
  FROM   a
 WHERE   EXISTS (SELECT   b.id
		   FROM   b
		  WHERE   b.condition = 'temp'
    AND   a.id = b.id)
```

## exp/imp failed as tablespace

exp/imp시 tablespace가 안맞는다고 에러날때 tablespaces 옵션을 줄수도 있지만 근본적으로 해당 schema(user)의 default tablespace를 맞춰주어야 한다.


```sql
--BDWORK User의 default tablespace 변경
SQL> ALTER USER bdwork DEFAULT tablespace ASBDARTBS;
 
USER altered.
 
SQL> SELECT username , default_tablespace FROM dba_users WHERE username IN ('WORK1','WORK2');
 
USERNAME                       DEFAULT_TABLESPACE
------------------------------ ------------------------------
WORK1                         ASBDARTBS
WORK2                         ASBDARTBS
```

## SQL Loader


```sql
LOAD DATA
INFILE 'part_master_201303_GEN.TXT'
APPEND
INTO TABLE PPRCTEMP
FIELDS TERMINATED BY '|'
(
    DIST_CD CHAR,
    PTNO CHAR,
    EFDT CHAR,
    ETDT CHAR,
    CURR_CD CHAR,
    PT_PRC INTEGER external,
    PRC_GRAD CHAR,
    PT_TYPE CHAR,
    NET_PRC INTEGER external,
    LIST_PRC INTEGER external,
    SUP_PTNO CHAR,
    ITC CHAR,
    USER_ID CHAR,
    REG_DT CHAR,
    CHG_DT "substr(:chg_dt,1,8)"
)
```

## GRANT


```sql
GRANT SELECT ON bdwork.tb_temp TO dl_wrty;
```

## 동적으로 쿼리 만들기


```sql
SELECT 'update wr_as_shop set bac_cd='''|| TRIM(bac_cd)||''' where shop_code='''|| shop_code||''';'
FROM wr_as_shop
WHERE TRIM(bac_cd) IS NOT NULL;
```

## update ~ select ~ join


```sql
UPDATE tb_temp
SET colat = (SELECT bac_cd
             FROM wr_as_shop
             WHERE colab = shop_type
             AND colac = shop_code
             AND colad = efdt
             AND colae = etdt
)
WHERE colaa = 'test';
```

## DB LINK 확인


```sql
SELECT * FROM sys.link$;
SELECT * FROM user_db_links;
SELECT * FROM all_db_links;
```

## Trigger Enable/Disable


```sql
ALTER TABLE p2partm0 enable ALL triggers;
```

## sh에서 sqlplus로 쿼리 돌리기


```sh
sqlplus id/pass @assign_accounts
```

## Number validation


```sql
col이 숫자나 점, 마이너스 기호로 시작하지 않는 놈들을 걸러낸다.
 
WHERE ltrim(col, '0123456789.-') IS NOT NULL;
```

## UNION, UNION ALL 등에서의 조건

- 컬럼의 형식은 동일해야 한다. (컬럼 타입, 개수)
- 컬럼의 이름은 달라도 된다. 컬럼 이름이 다를 경우, 맨 앞의 이름으로 통일된다.


## system계정에서 삭제된 데이터 파일 drop

시스템(system) 계정에서 먼저 삭제된 데이터 파일을 오프라인 드롭을 한다.


```sql
alter database datafile '[file path]' offline drop;
```


파일을 모두 드랍한다음에 논리적 테이블 스페이스를 삭제 하면 된다.


```sql
drop tablespace [테이블스페이스명] including contents and datafiles;
```


혹시 위의 경우가 아닌 서버 재부팅을 한뒤 shutdown 이나 startup이 안될경우 아래 방법참고


```sql
실수로 dbf 파일을 rm -rf 으로 먼저 삭제하고 drop tablespace 하면 셧다운이 안된다.
1. 데이터베이스 셧다운 : shutdown immediate;
2. 데이터베이스 마운트 : startup mount;
3. 테이블스페이스 오프라인 drop을 하자
alter database datafile '[file path]' offline drop;
4. 데이터베이스 오픈 : alter database open;

이렇게 한 다음에 논리적으로 테이블스페이스를 삭제하면 된다.
drop tablespace [테이블스페이스명] including contents and datafiles;
```

## SqlPlus에서 '&' 삽입하기

set define off;

위 명령어를 실행하면 &을 붙였을 때 변수로 선언하는 것이 아니라 문자로 들어간다.


## Export & Import


```
#!/bin/sh
 
EXP id/pass@ip:port/SID parfile=$1.par TABLES=$1 file=$1.dmp log=$1_exp.log statistics=NONE indexes=n constraints=n triggers=n buffer=100000000 compress=n grants=n
 
imp id/pass IGNORE=y commit=y file=$1.dmp log=$1_imp.log
 
#commit=y
```

## DB Tool에서 Procedure 호출하기

```sql
DECLARE
    OUOU VARCHAR2(20); -- OUTPUT 파라미터
BEGIN
    my_procedure(OUOU, '1', '2');
    DBMS_OUTPUT.PUT_LINE(OUOU);
END;
```

## 난수 문자열 생성하기

```sql
CREATE TABLE "TEST" 
   (  "CPNNO" VARCHAR2(50) NOT NULL ENABLE, 
   CONSTRAINT "TEST_PK" PRIMARY KEY ("CPNNO"));
CREATE OR REPLACE PROCEDURE P_TEST_CPNNO IS
  v_cpnNo VARCHAR2(50) := NULL;
  v_cpnCnt NUMBER := 0;
BEGIN
 
  FOR idx IN 1 .. 100000
  LOOP
 
    <<start_random>>
    SELECT 'AC'
           || DBMS_RANDOM.STRING('U', 1)
           || ROUND(DBMS_RANDOM.VALUE(0,9))
           || ROUND(DBMS_RANDOM.VALUE(0,9))
           || ROUND(DBMS_RANDOM.VALUE(0,9))
           || ROUND(DBMS_RANDOM.VALUE(0,9))
           || ROUND(DBMS_RANDOM.VALUE(0,9))
           || ROUND(DBMS_RANDOM.VALUE(0,9))
           || 'I'
      INTO v_cpnNo
      FROM DUAL;
 
    SELECT COUNT(*)
      INTO v_cpnCnt
      FROM TEST
     WHERE cpnNo = v_cpnNo;
 
    IF v_cpnCnt <= 0
    THEN
      INSERT INTO TEST (cpnNo)
           VALUES (v_cpnNo);
    ELSE
        GOTO start_random;
    END IF;
  END LOOP;
 
  COMMIT;
 
  EXCEPTION
     WHEN OTHERS THEN
       DBMS_OUTPUT.PUT_LINE('ERROR : ' || SQLCODE );
       DBMS_OUTPUT.PUT_LINE('SQLERRM : ' || SUBSTR(SQLERRM, 1, 200) );
       ROLLBACK;
END;
BEGIN
  P_TEST_CPNNO();
END;
```

## Sequence CURRVAL, NEXTVAL 값 조정 및 초기화

```sql
-- 시퀀스의 현재 값을 확인
SELECT LAST_NUMBER FROM USER_SEQUENCES WHERE SEQUENCE_NAME = 'TB_ZZTRACE_SQ01';

-- 시퀀스의 INCREMENT 를 현재 값만큼 빼도록 설정 (아래는 현재값이 999999 일 경우)
ALTER SEQUENCE TB_ZZTRACE_SQ01 INCREMENT BY -999999;

-- 시퀀스에서 다음 값을 가져 온다
SELECT TB_ZZTRACE_SQ01.NEXTVAL FROM DUAL;

-- 현재 값을 확인 해보면 -999999 만큼 증가 했다
SELECT TB_ZZTRACE_SQ01.CURRVAL FROM DUAL;

-- 시퀀스의 INCREMENT 를 1로 설정 한다
ALTER SEQUENCE TB_ZZTRACE_SQ01 INCREMENT BY 1;

-- 시퀀스가 1부터 다시 시작 한다.
```



# Error

## ORA-04091 : Mutation error

Oracle은 Table에 row Trigger를 만들면 원천적으로 해당 Table을 아예 Access할수 없게 하며 이와 같은 원칙에 위배될때 mutationg error가 발생하게 된다. 즉, 트리거 안에서는 자기 자신의 테이블을 접근할 수가 없다. 

해결책은... 대신 :OLD 혹은 :NEW로 ROW 단위 접근이 가능하므로 이걸 사용하거나 임시 테이블 사용, :OLD와 :NEW를 사용하지 않는 Statement trigger, PL/SQL 테이블 생성 사용 등이 있다.

## ORA-00911:문자가 부적합 합니다.

쿼리문을 짤 때 Application 내에서는 ; 문자를 빼고 Statement를 작성해야 한다.

## LONG 값은 LONG 열에만 입력

java.sql.SQLException: ORA-01461: LONG 값은 LONG 열에만 입력할 수 있습니다 관련
위와 같은 에러를 발견했다.

char-set 이 euc-kr로 되어 있던 파일을 읽어 utf8의 케릭터 셋의 오라클에 preparestatement로 입력하는 도중 만남 Exception이다.

insert하려는 데이타의 타입은 varchar2(4000) 이었고, 실제 euc-kr의 길이는 3900이었으나, char-set변경시 그 길이가 넘어가 생긴 에러였다. 해결은....CLOB으로 변경하여 처리했음