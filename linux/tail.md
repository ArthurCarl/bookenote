# Tail
输出文件的尾部

语法:
```
tail [OPTION]... [FILE]...q
```

介绍一些比较有用的参数:
1. `--max-unchanged-stats=N` 在N次迭代后，文件大小没有改变则重新打开文件，与 `--follow=name` 一起使用
2. `--pid=PID` `PID`进程死亡则停止终结
3. `--retry` 当文件不能被访问时不断重试访问文件,`--follow=name` 一起使用特别有用
4. `-s,--sleep-interval=N` 每隔N秒则迭代一次



