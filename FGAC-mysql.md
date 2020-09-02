**This procedure already verified in Aurora MySQL 5.7.2081**

# FGAC Case 1 :

**Referenced by http://www.jeromeradix.com/2006/12/mysql-50-fine-grained-access-control.html**  
**Some steps were changed due to syntax error and aurora engine characteristics**

**Create database fgac, applidata and Create user user1**

```
kiwony@kiwonymac.com:/Users/kiwony> mysql -uadmin -haurora-mysql-572081.cluster-ml2se7ljk25g.ap-northeast-2.rds.amazonaws.com -p

mysql> create database applidata;
mysql> create user user1 identified by '<PASSWORD>';
mysql> grant select, insert, update, delete on applidata.\* to user1@'%';
mysql> flush privileges;
mysql> create database fgac;
mysql> use fgac;
```

**Create user info table to filter data**

```
mysql> create table my_users (username varchar(81) primary key, password varchar(81));
mysql> insert into my_users (username, password) values ('userA', 'userA');
mysql> insert into my_users (username, password) values ('userB', 'userB');
mysql> insert into my_users (username, password) values ('userC', 'userC');
mysql> insert into my_users (username, password) values ('userD', 'userD');
```

**Create Credit card history table**

```
mysql> create table credit_cards(num int primary key, expire_date date, owner varchar(81));
mysql> insert into credit_cards(num, expire_date, owner) values (1234, NOW(), 'userA');
mysql> insert into credit_cards(num, expire_date, owner) values (5678, NOW(), 'userB');

mysql> select \* from credit_cards;
+------+-------------+-------+
| num | expire_date | owner |
+------+-------------+-------+
| 1234 | 2020-08-09 | userA |
| 5678 | 2020-08-09 | userB |
+------+-------------+-------+
2 rows in set (0.01 sec)
```

**Create functions getusername, getpassword to retrieve**

```
delimiter //

drop function getusername;
//

create function getusername() returns varchar(81)
begin
return @name;
end;
//

drop function getpassword;
//

create function getpassword() returns varchar(81)
begin
return @pwd;
end;
//

create or replace view applidata.credit_cards as
select num, expire_date
from fgac.credit_cards, fgac.my_users
where username = fgac.getusername()
and password = fgac.getpassword()
and owner = username;
//
```

**Check applidata without username and password**

```
mysql> use applidata
mysql> show tables;
-> //
+---------------------+
| Tables_in_applidata |
+---------------------+
| credit_cards |
+---------------------+
1 row in set (0.01 sec)

mysql> select \* from credit_cards;
Empty set (0.01 sec)
```

**Set username & password to use our view**

```
mysql> set @name = 'userA';

mysql> set @pwd = 'userA';

mysql> select \* from credit_cards;
+------+-------------+
| num | expire_date |
+------+-------------+
| 1234 | 2020-08-09 |
+------+-------------+
1 row in set (0.01 sec)

mysql> set @name='userB';
Query OK, 0 rows affected (0.02 sec)

mysql> set @pwd='UserB';
Query OK, 0 rows affected (0.01 sec)

mysql> select \* from credit_cards;
+------+-------------+
| num | expire_date |
+------+-------------+
| 5678 | 2020-08-09 |
+------+-------------+
1 row in set (0.01 sec)
```

**FGAC with IAM user**

```
mysql> create user userA identified with AWSAuthenticationPlugin AS 'RDS';
Query OK, 0 rows affected (0.02 sec)

mysql> grant all on applidata.\* to 'userA'@'%';
Query OK, 0 rows affected (0.01 sec)

kiwony@kiwonymac.com:/Users/kiwony> DBUSER=userA
kiwony@kiwonymac.com:/Users/kiwony> TOKEN="$(aws rds generate-db-auth-token --hostname $RDSHOST --port 3306 --region ap-northeast-2 --username $DBUSER )"
kiwony@kiwonymac.com:/Users/kiwony> mysql --host=$RDSHOST --port=3306 --ssl-ca=rds-combined-ca-bundle.pem --enable-cleartext-plugin --user=$DBUSER --password=$TOKEN

kiwony@kiwonymac.com:/Users/kiwony> mysql --host=$RDSHOST --port=3306 --ssl-ca=rds-combined-ca-bundle.pem --enable-cleartext-plugin --user=$DBUSER --password=$TOKEN -e "Set @name='$DBUSER';set @pwd='UserA';select \* from credit_cards" applidata
mysql: [Warning] Using a password on the command line interface can be insecure.
+------+-------------+
| num | expire_date |
+------+-------------+
| 1234 | 2020-08-09 |
+------+-------------+

```
