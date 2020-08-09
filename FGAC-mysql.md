**This procedure already verified in Aurora MySQL 5.7.2081**

# FAGC Case 1 :

**Referenced by http://www.jeromeradix.com/2006/12/mysql-50-fine-grained-access-control.html**
**Some steps were changed due to syntax error and aurora engine characteristics**

mysql> create database applidata;
Query OK, 1 row affected (0.03 sec)

mysql> create user user1 identified by 'user1';
Query OK, 0 rows affected (0.04 sec)

mysql> grant select, insert, update, delete on applidata.\* to user1@'%';
Query OK, 0 rows affected (0.03 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)

mysql> create database fgac;
Query OK, 1 row affected (0.03 sec)

mysql> use fgac;
Database changed

mysql> create table my_users (username varchar(81) primary key, password varchar(81));
Query OK, 0 rows affected (0.04 sec)
