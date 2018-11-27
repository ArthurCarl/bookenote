# grep

## 输出行数控制

```
grep -C 10 Micky dir/file.md # Mick的上下10行
grep -B 10 Micky dir/file.md # Mick的上10行
grep -A 10 Micky dir/file.md # Mick的下10行
```

```
grep [OPTIONS] PATTERN [FILE...]
```

## 匹配 A或B

```
grep -E --color=auto -i 'A|B' file # -E 扩展正则，可以识别`A|B` 的正则
egrep --color=auto 'A|B' # 与上等效果
```

