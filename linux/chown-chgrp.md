# chgrp

改变文件的用户组:
```
chgrp [OPTION]... GROUP FILE...
------
chgrp staff /u   ##修改`/u`的用户组为`staff`
chgrp -hR staff  ##修改`/u`及子目录中文件的用户组为 `staff`q



```
# chown
改变文件所有者:
```
chown [OPTION]... [OWNER][:[GROUP]] FILE..
------
chown root:staff /u
chown -hR root /u
```

# chmod
修改文件权限:
```
chmod [OPTION]... MODE[,MODE]... FILE...
chmod [OPTION]... OCTAL_MODE FILE
```
r -4(读)  
w -2(写)  
x -1(执行)  


```
chmod 777 /homw
chmod u+r,g+w,o+x /home
chmod u=rw,g=rx,o=rx /home
```


