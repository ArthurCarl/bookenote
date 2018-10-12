## NETWORK/ADMIN/tnsnames.ora

```
AliasName = 
  (DESCRIPTION=
    (ADDRESS_LIST=
      (ADDRESS=(PROTOCOL=TCP)(HOST=127.0.0.1)(PORT=1521))
    )
    (CONNECT_DATA=
      (SERVICE_NAME = 实例名称)
    )
  )
```



## 批量更新存储过程:
```sql
declare 
  i_maxrows number := 5000;
  cursor cur is 
    select t.rowid,t.id
      from t_table1 t where t.a is not null;
  type t_record is record(t_rowid varchar2(80),t_id varchar2(10));
  type t_list is table of t_record;
  v_temp_table t_list;
begin
  open cur;
  loop
    exit when cur%/notfound;
    
    fetch cur bulk collect into v_temp_table limit i_maxrows;
    for i in 1 .. v_temp_table.count loop
      update t_table1 set a = 'a' where rowid = v_temp_table(i).t_rowid;
    end loop;
    commit;
  end loop;
  close cur;
end;
/
```
