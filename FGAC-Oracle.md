
# Fine Grained Access Control

## Definition
FGAC makes specific user is only able to access specific data or column. 
It provides users transparently with only the data they need, so we can implement multi tenancy easily using it.
It is like user's own database, even it is one database or one table.

```
oracle@oracle11g:/home/oracle> ss
SQL> grant create any context to oshop;

Grant succeeded.

SQL> grant execute on dbms_session to oshop;

Grant succeeded.
```