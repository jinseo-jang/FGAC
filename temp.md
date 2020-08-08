

# FGAC Case 2 : Row Level FGAC : Hiding specific data based on row data such 
## This is working for OnPrem oracle and RDS oracle

**Create Data Schema - octank, FGAC Schema - emp_admin**
```
oracle@oracle11g:/home/oracle> sqlplus admin/Octank#1234@rds
create user octank identified by Octank#1234 default tablespace users temporary tablespace temp quota unlimited on users;
grant connect,resource to octank;

create user emp_admin identified by Octank#1234 default tablespace users temporary tablespace temp quota unlimited on users;
grant connect,resource to emp_admin;
grant create session, create any context, create procedure, create trigger, administer database trigger to emp_admin;
grant execute on dbms_session to emp_admin;
grant execute on dbms_rls to emp_admin;
```

**Create sample data in octank schema**
```
oracle@oracle11g:/home/oracle> sqlplus octank/Octank#1234@rds
SQL> @demobld.sql

create table mapping as select EMPNO,ENAME from emp order by 1;

SQL> grant select on mapping to emp_admin;

```

**Create oracle schemas based on emp table**
```
oracle@oracle11g:/home/oracle> sqlplus admin/Octank#1234@rds

SQL> select 'create user ' || ENAME || ' identified by Octank#1234 default tablespace users temporary tablespace temp quota unlimited on users;' as Query from emp order by empno;

QUERY
-----------------------------------------------------------------------------------------------------------------------------
create user SMITH identified by Octank#1234 default tablespace users temporary tablespace temp quota unlimited on users;
create user ALLEN identified by Octank#1234 default tablespace users temporary tablespace temp quota unlimited on users;
create user WARD identified by Octank#1234 default tablespace users temporary tablespace temp quota unlimited on users;
create user JONES identified by Octank#1234 default tablespace users temporary tablespace temp quota unlimited on users;
create user MARTIN identified by Octank#1234 default tablespace users temporary tablespace temp quota unlimited on users;
create user BLAKE identified by Octank#1234 default tablespace users temporary tablespace temp quota unlimited on users;
create user CLARK identified by Octank#1234 default tablespace users temporary tablespace temp quota unlimited on users;
create user SCOTT identified by Octank#1234 default tablespace users temporary tablespace temp quota unlimited on users;
create user KING identified by Octank#1234 default tablespace users temporary tablespace temp quota unlimited on users;
create user TURNER identified by Octank#1234 default tablespace users temporary tablespace temp quota unlimited on users;
create user ADAMS identified by Octank#1234 default tablespace users temporary tablespace temp quota unlimited on users;
create user JAMES identified by Octank#1234 default tablespace users temporary tablespace temp quota unlimited on users;
create user FORD identified by Octank#1234 default tablespace users temporary tablespace temp quota unlimited on users;
create user MILLER identified by Octank#1234 default tablespace users temporary tablespace temp quota unlimited on users;

14 rows selected.

SQL> select 'grant create session to ' || ENAME || ';' as GrantQuery from emp order by empno;

GRANTQUERY
-----------------------------------
grant create session to SMITH;
grant create session to ALLEN;
grant create session to WARD;
grant create session to JONES;
grant create session to MARTIN;
grant create session to BLAKE;
grant create session to CLARK;
grant create session to SCOTT;
grant create session to KING;
grant create session to TURNER;
grant create session to ADAMS;
grant create session to JAMES;
grant create session to FORD;
grant create session to MILLER;

14 rows selected.

SQL> select 'grant select on octank.mapping to ' || ename || ';' as GrantQry2 from octank.emp order by 1;

GRANTQRY2
---------------------------------------------
grant select on octank.mapping to ADAMS;
grant select on octank.mapping to ALLEN;
grant select on octank.mapping to BLAKE;
grant select on octank.mapping to CLARK;
grant select on octank.mapping to FORD;
grant select on octank.mapping to JAMES;
grant select on octank.mapping to JONES;
grant select on octank.mapping to KING;
grant select on octank.mapping to MARTIN;
grant select on octank.mapping to MILLER;
grant select on octank.mapping to SCOTT;
grant select on octank.mapping to SMITH;
grant select on octank.mapping to TURNER;
grant select on octank.mapping to WARD;

14 rows selected.

SQL> select 'grant select on octank.emp  to ' || ename || ';' as GrantQry2 from octank.emp order by 1;

GRANTQRY2
------------------------------------------
grant select on octank.emp  to ADAMS;
grant select on octank.emp  to ALLEN;
grant select on octank.emp  to BLAKE;
grant select on octank.emp  to CLARK;
grant select on octank.emp  to FORD;
grant select on octank.emp  to JAMES;
grant select on octank.emp  to JONES;
grant select on octank.emp  to KING;
grant select on octank.emp  to MARTIN;
grant select on octank.emp  to MILLER;
grant select on octank.emp  to SCOTT;
grant select on octank.emp  to SMITH;
grant select on octank.emp  to TURNER;
grant select on octank.emp  to WARD;

14 rows selected.


```



**grant select on octank.emp to emp_admin to create mapping table**
```
SQL> grant select on octank.emp to emp_admin;

Grant succeeded.
```

**Create user emp_admin who creates procedure & trigger to limit data access from users**
```
oracle@oracle11g:/home/oracle> sqlplus emp_admin/Octank#1234@rds

create or replace context empno_ctx using empno_ctx_pkg;

create or replace package empno_ctx_pkg
is
procedure set_empno;
end;
/

create or replace package body empno_ctx_pkg
is
    procedure set_empno
    as
    emp_no NUMBER;
begin
    select empno into emp_no from octank.mapping where ename=SYS_CONTEXT('USERENV', 'SESSION_USER');
    DBMS_SESSION.SET_CONTEXT('empno_ctx','empno_attr',emp_no);
    exception
        when no_data_found then null;
    end set_empno;
END;
/
SQL> grant execute on empno_ctx_pkg to admin;

Grant succeeded.

drop trigger set_empno_ctx_trigger;
create trigger set_empno_ctx_trigger after logon on database
begin
    emp_admin.empno_ctx_pkg.set_empno;
END;
/

```

**User defined context check using by SMITH**
```
oracle@oracle11g:/home/oracle> sqlplus SMITH/Octank#1234@rds
SQL> SELECT SYS_CONTEXT ('empno_ctx', 'empno_attr') empno_attrib FROM DUAL;

EMPNO_ATTRIB
---------------
7369


**Create policy & add**
oracle@oracle11g:/home/oracle> sqlplus emp_admin/Octank#1234@rds

CREATE OR REPLACE FUNCTION get_emp_info(
schema_p IN VARCHAR2,
table_p IN VARCHAR2)
RETURN VARCHAR2
AS
emp_pred VARCHAR2 (400);
BEGIN
emp_pred := 'empno = SYS_CONTEXT(''empno_ctx'', ''empno_attr'')';
RETURN emp_pred;
END;
/



BEGIN
DBMS_RLS.ADD_POLICY (
object_schema => 'octank',
object_name => 'emp',
policy_name => 'emp_policy',
function_schema => 'emp_admin',
policy_function => 'get_emp_info',
statement_types => 'select');
END;
/
```

**Check row level FGAC is working or not**
```
oracle@oracle11g:/home/oracle> sqlplus SMITH/Octank#1234@rds

SQL> select * from octank.emp;

     EMPNO ENAME      JOB	       MGR HIREDATE	    SAL       COMM     DEPTNO
---------- ---------- --------- ---------- --------- ---------- ---------- ----------
      7369 SMITH      CLERK	      7902 17-DEC-80	    800 		   20

oracle@oracle11g:/home/oracle> sqlplus ALLEN/Octank#1234@rds
SQL> select * from octank.emp;

     EMPNO ENAME      JOB	       MGR HIREDATE	    SAL       COMM     DEPTNO
---------- ---------- --------- ---------- --------- ---------- ---------- ----------
      7499 ALLEN      SALESMAN	      7698 20-FEB-81	   1600        300	   30