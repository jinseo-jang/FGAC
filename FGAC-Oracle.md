
# Fine Grained Access Control

## Definition
FGAC makes specific user is only able to access specific data or column.  
It provides users transparently with only the data they need, so we can implement multi tenancy easily using it.  
Even though there is one database or one table, users are able to using like they have their own database using FGAC.


## Benefits
1. Security aspect 
In Oracle, there is DB vault option.  
DB vault provides lots of security features and one of them is limiting data access privileges based on rule.  
You can define rule to limit access to specific table or data using DB vault.  
So even if DABs have sys or system account, they are not able to access different schema's data because of DB vault.  
But you need DB vault license to use the option.   
You are able to limit data access without DB vault feature by FGAC.   
Once user connection is established, user application have no way to avoid FGAC. So you can easily limit data access using FGAC.

2. Simplicy 
You can set data access rule using FGAC, and application developers don't need to care about their logic to implement limiting data access.


# FGAC Case 1 : Column Level FGAC : Hiding specific data based on important column such as salary, SSN
## This is working for OnPrem oracle and RDS oracle

```
SQL> create user user1 identified by Octank#1234 default tablespace users temporary tablespace temp quota unlimited on users;
SQL> grant connect,resource to user1;
SQL> grant create session, create any context, create procedure, create trigger, administer database trigger to user1;
SQL> grant execute on dbms_session to user1;
SQL> grant execute on dbms_rls to user1;
```

**Connect to database using user1 schema**
```
oracle@oracle11g:/home/oracle> sqlplus user1
SQL> demobld.sql

```
**user1 is able to select all row in emp table**
```
SQL> select * from emp order by empno order by 1;
     EMPNO ENAME      JOB	       MGR HIREDATE	    SAL       COMM     DEPTNO
---------- ---------- --------- ---------- --------- ---------- ---------- ----------
      7369 SMITH      CLERK	      7902 17-DEC-80	    800 		   20
      7499 ALLEN      SALESMAN	      7698 20-FEB-81	   1600        300	   30
      7521 WARD       SALESMAN	      7698 22-FEB-81	   1250        500	   30
      7566 JONES      MANAGER	      7839 02-APR-81	   2975 		   20
      7654 MARTIN     SALESMAN	      7698 28-SEP-81	   1250       1400	   30
      7698 BLAKE      MANAGER	      7839 01-MAY-81	   2850 		   30
      7782 CLARK      MANAGER	      7839 09-JUN-81	   2450 		   10
      7788 SCOTT      ANALYST	      7566 09-DEC-82	   3000 		   20
      7839 KING       PRESIDENT 	   17-NOV-81	   5000 		   10
      7844 TURNER     SALESMAN	      7698 08-SEP-81	   1500 	 0	   30
      7876 ADAMS      CLERK	      7788 12-JAN-83	   1100 		   20
      7900 JAMES      CLERK	      7698 03-DEC-81	    950 		   30
      7902 FORD       ANALYST	      7566 03-DEC-81	   3000 		   20
      7934 MILLER     CLERK	      7782 23-JAN-82	   1300 		   10
14 rows selected.
```

**https://docs.oracle.com/en/database/oracle/oracle-database/19/arpls/DBMS_RLS.html#GUID-1E528A51-DE53-4961-8770-C53924E427CC**

**Create Function to add where condition that only shows JOB in ('CLERK','SALESMAN')**
```
SQL> create or replace function hide_sal_comm(
    v_schema in varchar2,
    v_objname in varchar2)

    return varchar2 as
    con varchar2 (200);

    begin
        con := 'JOB in (''CLERK'',''SALESMAN'')';
        return(con);
        END hide_sal_comm;
/
Function created.
```

**Add Policy into schema.table, if user select column SAL or COMM, then hide_sal_comm function filtering the result**
```
SQL> exec DBMS_RLS.ADD_POLICY ('user1', 'emp', 'emp_policy', 'user1', 'hide_sal_comm', 'select',false,true,false,null,false,'SAL,COMM');
PL/SQL procedure successfully completed.
```

**Same "select * from emp;" query shows result as JOB=CLERK or JOB=SALESMAN**
```
SQL> select * from emp;

     EMPNO ENAME      JOB	       MGR HIREDATE	    SAL       COMM     DEPTNO
---------- ---------- --------- ---------- --------- ---------- ---------- ----------
      7369 SMITH      CLERK	      7902 17-DEC-80	    800 		   20
      7499 ALLEN      SALESMAN	      7698 20-FEB-81	   1600        300	   30
      7521 WARD       SALESMAN	      7698 22-FEB-81	   1250        500	   30
      7654 MARTIN     SALESMAN	      7698 28-SEP-81	   1250       1400	   30
      7844 TURNER     SALESMAN	      7698 08-SEP-81	   1500 	 0	   30
      7876 ADAMS      CLERK	      7788 12-JAN-83	   1100 		   20
      7900 JAMES      CLERK	      7698 03-DEC-81	    950 		   30
      7934 MILLER     CLERK	      7782 23-JAN-82	   1300 		   10
8 rows selected.
```

**If there is no select on SAL,COMM column, then shows all datas**
```
SQL> select EMPNO, ENAME,JOB,MGR,HIREDATE,DEPTNO from EMP order by 1;

     EMPNO ENAME      JOB	       MGR HIREDATE	 DEPTNO
---------- ---------- --------- ---------- --------- ----------
      7369 SMITH      CLERK	      7902 17-DEC-80	     20
      7499 ALLEN      SALESMAN	      7698 20-FEB-81	     30
      7521 WARD       SALESMAN	      7698 22-FEB-81	     30
      7566 JONES      MANAGER	      7839 02-APR-81	     20
      7654 MARTIN     SALESMAN	      7698 28-SEP-81	     30
      7698 BLAKE      MANAGER	      7839 01-MAY-81	     30
      7782 CLARK      MANAGER	      7839 09-JUN-81	     10
      7788 SCOTT      ANALYST	      7566 09-DEC-82	     20
      7839 KING       PRESIDENT 	   17-NOV-81	     10
      7844 TURNER     SALESMAN	      7698 08-SEP-81	     30
      7876 ADAMS      CLERK	      7788 12-JAN-83	     20
      7900 JAMES      CLERK	      7698 03-DEC-81	     30
      7902 FORD       ANALYST	      7566 03-DEC-81	     20
      7934 MILLER     CLERK	      7782 23-JAN-82	     10
14 rows selected.
```


**drop poilcy procedure**
```
SQL> exec dbms_rls.drop_policy('user1','emp','emp_policy');



```


# FGAC Case 2 : Row Level FGAC : Hiding specific data based on row data such 
## This is working for OnPrem oracle and RDS oracle

```
SQL> create user user1 identified by Octank#1234 default tablespace users temporary tablespace temp quota unlimited on users;
SQL> grant connect,resource to user1;
SQL> grant create session, create any context, create procedure, create trigger, administer database trigger to user1;
SQL> grant execute on dbms_session to user1;
SQL> grant execute on dbms_rls to user1;
```

**Connect to database using user1 schema**
```
oracle@oracle11g:/home/oracle> sqlplus user1
SQL> demobld.sql

```
**user1 is able to select all row in emp table**
```
SQL> select * from emp order by empno order by 1;
     EMPNO ENAME      JOB	       MGR HIREDATE	    SAL       COMM     DEPTNO
---------- ---------- --------- ---------- --------- ---------- ---------- ----------
      7369 SMITH      CLERK	      7902 17-DEC-80	    800 		   20
      7499 ALLEN      SALESMAN	      7698 20-FEB-81	   1600        300	   30
      7521 WARD       SALESMAN	      7698 22-FEB-81	   1250        500	   30
      7566 JONES      MANAGER	      7839 02-APR-81	   2975 		   20
      7654 MARTIN     SALESMAN	      7698 28-SEP-81	   1250       1400	   30
      7698 BLAKE      MANAGER	      7839 01-MAY-81	   2850 		   30
      7782 CLARK      MANAGER	      7839 09-JUN-81	   2450 		   10
      7788 SCOTT      ANALYST	      7566 09-DEC-82	   3000 		   20
      7839 KING       PRESIDENT 	   17-NOV-81	   5000 		   10
      7844 TURNER     SALESMAN	      7698 08-SEP-81	   1500 	 0	   30
      7876 ADAMS      CLERK	      7788 12-JAN-83	   1100 		   20
      7900 JAMES      CLERK	      7698 03-DEC-81	    950 		   30
      7902 FORD       ANALYST	      7566 03-DEC-81	   3000 		   20
      7934 MILLER     CLERK	      7782 23-JAN-82	   1300 		   10
14 rows selected.
```

**https://docs.oracle.com/en/database/oracle/oracle-database/19/arpls/DBMS_RLS.html#GUID-1E528A51-DE53-4961-8770-C53924E427CC**

**Create Function to add where condition that only shows JOB in ('CLERK','SALESMAN')**
```
SQL> create or replace function hide_sal_comm(
    v_schema in varchar2,
    v_objname in varchar2)

    return varchar2 as
    con varchar2 (200);

    begin
        con := 'JOB in (''CLERK'',''SALESMAN'')';
        return(con);
        END hide_sal_comm;
/
Function created.
```

**Add Policy into schema.table, if user select column SAL or COMM, then hide_sal_comm function filtering the result**
```
SQL> exec DBMS_RLS.ADD_POLICY ('user1', 'emp', 'emp_policy', 'user1', 'hide_sal_comm', 'select',false,true,false,null,false,'SAL,COMM');
PL/SQL procedure successfully completed.
```

**Same "select * from emp;" query shows result as JOB=CLERK or JOB=SALESMAN**
```
SQL> select * from emp;

     EMPNO ENAME      JOB	       MGR HIREDATE	    SAL       COMM     DEPTNO
---------- ---------- --------- ---------- --------- ---------- ---------- ----------
      7369 SMITH      CLERK	      7902 17-DEC-80	    800 		   20
      7499 ALLEN      SALESMAN	      7698 20-FEB-81	   1600        300	   30
      7521 WARD       SALESMAN	      7698 22-FEB-81	   1250        500	   30
      7654 MARTIN     SALESMAN	      7698 28-SEP-81	   1250       1400	   30
      7844 TURNER     SALESMAN	      7698 08-SEP-81	   1500 	 0	   30
      7876 ADAMS      CLERK	      7788 12-JAN-83	   1100 		   20
      7900 JAMES      CLERK	      7698 03-DEC-81	    950 		   30
      7934 MILLER     CLERK	      7782 23-JAN-82	   1300 		   10
8 rows selected.
```

**If there is no select on SAL,COMM column, then shows all datas**
```
SQL> select EMPNO, ENAME,JOB,MGR,HIREDATE,DEPTNO from EMP order by 1;

     EMPNO ENAME      JOB	       MGR HIREDATE	 DEPTNO
---------- ---------- --------- ---------- --------- ----------
      7369 SMITH      CLERK	      7902 17-DEC-80	     20
      7499 ALLEN      SALESMAN	      7698 20-FEB-81	     30
      7521 WARD       SALESMAN	      7698 22-FEB-81	     30
      7566 JONES      MANAGER	      7839 02-APR-81	     20
      7654 MARTIN     SALESMAN	      7698 28-SEP-81	     30
      7698 BLAKE      MANAGER	      7839 01-MAY-81	     30
      7782 CLARK      MANAGER	      7839 09-JUN-81	     10
      7788 SCOTT      ANALYST	      7566 09-DEC-82	     20
      7839 KING       PRESIDENT 	   17-NOV-81	     10
      7844 TURNER     SALESMAN	      7698 08-SEP-81	     30
      7876 ADAMS      CLERK	      7788 12-JAN-83	     20
      7900 JAMES      CLERK	      7698 03-DEC-81	     30
      7902 FORD       ANALYST	      7566 03-DEC-81	     20
      7934 MILLER     CLERK	      7782 23-JAN-82	     10
14 rows selected.
```


**drop poilcy procedure**
```
SQL> exec dbms_rls.drop_policy('user1','emp','emp_policy');



```