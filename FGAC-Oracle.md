
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



```
SQL> @createuser
Enter value for username: user1
old   1: create user &username identified by Octank#1234 default tablespace users temporary tablespace temp quota unlimited on users
new   1: create user user1 identified by Octank#1234 default tablespace users temporary tablespace temp quota unlimited on users

User created.

old   1: grant connect,resource to &username
new   1: grant connect,resource to user1

Grant succeeded.


SQL> grant create session, create any context, create procedure, create trigger, administer database trigger to user1;

SQL> grant execute on dbms_session to user1;

SQL> grant execute on dbms_rls to user1;

oracle@oracle11g:/home/oracle> sqlplus user1

```
**user1 is able to select all row in emp table**
```
SQL> select * from emp order by empno;
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




SQL> create or replace function hide_sal_comm(
    v_schema in varchar2,
    v_objname in varchar2)

    return varchar2 as
    con varchar2 (200);

    begin
        con := 'deptno=30';
        return(con);
        END hide_sal_comm;
/
Function created.


SQL> exec DBMS_RLS.ADD_POLICY ('user1', 'emp', 'emp_policy', 'user1', 'hide_sal_comm', 'select',false,true,false,null,false,'SAL,COMM');
PL/SQL procedure successfully completed.




SQL> exec dbms_rls.drop_policy('user1','emp','emp_policy');



SQL> exec DBMS_RLS.ADD_POLICY ('user1', 'emp', 'emp_policy', 'user1', 'hide_sal_comm', 'select',false,true,false,null,false,'SAL,COMM');

PL/SQL procedure successfully completed.


```






```
oracle@oracle11g:/home/oracle> ss
SQL> grant create any context to oshop;

Grant succeeded.

SQL> grant execute on dbms_session to oshop;

Grant succeeded.
```