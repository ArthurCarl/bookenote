# AWK

### Input

#### `RS` -行分隔符  

``` shell
$ awk 'BEGIN {RS = "U"} {print $)} ' mail-list

$ awk '{ print $0 }' RS="u" mail-list

# RS = reg
$ echo record 1 AAAA record 2 BBBB record 3 |
> gawk 'BEGIN { RS = "\n|( *[[:upper:]]+ *)" }
>			  { print "Record =", $0,"and RT = [" RT "]" }'	
```

#### Fields  

``` shell
$ awk ' $1 ~ /li/ { print $0 } ' mail-list

$ awk ' /li/ { print $1, $NF } ' mail-list

$ awk '{ nboxes = $3 ; $3 = $3 - 10 } { print nboxes, $3 }' inventory-shipped

$ awk ' { $2 = $2 -10; print $0 } ' inventory-shipped

$ awk '{ $6 = ($5 + $4 + $3 + $2) } { print $6 }' inventory-shipped

$ echo 'John Q. Smith, 29 Oak St., Walamazoo, MI 42139' | awk 'BEGIN { FS = "," }; { print $2 }'

$ echo ' a b c d ' | awk 'BEGIN { FS = "[ \t\n]+" } { print $2 }'  
a

$ echo '	a b c d' | awk '{ print ; $2 = $2; print }'  
    a b c d
a b c d

# 把一行当为一个字段
$ awk -F'\n' 'program' files …

```

#### Fixed-Width Data

#### FPAT



### Output

``` shell
$ awk 'BEGIN { print "Month Creates"; print "------ ------" } {print $1, $2}' inventory-shipped

# OFS = out feild seperator  ORS = out raw seperator
$ awk 'BEGIN { OFS = ";"; ORS = "\n\n" } { print $1, $2 }' mail-list

$ awk 'BEGIN {OFMT = "%.0f" print 17.23, 17.54}


```

#### `printf`

```shell
$ awk 'BEGIN { \
> ORS = "\nOUCH!\n"; OFS = "+";\
> msg = "Don\47t Panic!";\
> printf "%s\n", msg\
> }'
Don't Panic!

$ awk 'BEGIN { printf "%4.3e\n", 1950}'

# N$ = 特定参数位置
$ awk 'BEGIN {prinf "%2$s %1$s\n", "panic", "don\47t"}'


$ awk '{ printf "%-10s %s\n", $1, $2 }' mail-list

$ awk '{print $2 > "phone-list"; print $1 > "name-list"}' mail-list

$ awk '{print $1 > "names.unsorted"; command="sort -r > names.sorted"; print $1 | command}' mail-list
```

`/dev/stdin` 	-标准输入流(file descriptor 0)  
`/dev/stdout` -标准输出流(file descriptor 1)  
`/dev/stderr`	-标准错误输出流(file descriptor 2)  

#### close Input and Output Redirections

程序最后需要关闭流

``` shell
$  "sort -r names" | getline foo
	......

$ close("sort -r names")
```

### Expressions

``` bash
$ gawk 'BEGIN { printf "%d, %d, %d\n", 011, 11, 0x11 }'

$ awk '{print $n}' n=4 inventory-shipped n=2 mail-list
```

### Patterns, Actions, and Variables

#### Range Patterns

```
$ vim myfile
on my off
off you on
hello I don't konw.
on it is true
off turn off

$ awk '$1 == "on", $1 == "off"' myfile
on my off
off you on
on it is true
off turn off
```

#### BEGIN/END

```
$ awk '\
> BEGIN {print "Analysis of \"li\"" }\
> /li/ {++n}\
> END {print "\"li\" appears in", n, "records." }' mail-list

Analysis of "li"
"li" appears in 4 records.
```

#### Patterns Action

```
$ awk 'BEGIN { \
	for (x = 0; x <= 20; x++) { \
		if (x == 5){ \
			continue ;\
		}\
		printf "%d ", x ;\
	}\
	print ""\
}'
```

#### Array loopchek.awk

``` awk
$ vim loopchek.awk
BEGIN {
	a["here"] = "here";
	a["is"] = "is";
	a["a"] = "a";
	a["loop"] = "loop";
	for (i in a) {
		j++;
		a[j] = j;
		print a[i];
		print i;
	}
	print a;
}
here
here
loop
loop
a
a
is
is
```

#### Using Predefined Array Scanning Orders with gawk

``` awk
$ gawk ' \
>    BEGIN { \
>	   PROCINFO["sorted_in"] = "@ind_str_asc";\
>      a[4] = 4;\
>      a[3] = 3;\
>      for( i in a){ \
>        print i,a[i];\
>	   }' 
```

##### Multidimensional Array

``` awk
$ vim multidimensional.awk
{
	if (max_nf < NF){
		max_nf = NF;
	}
	max_nr = NR;
	for (x = 1; x <= NF; x++){
		vector[x, NR] = $x;
	}
}
END {
	for (x = 1; x <= max_nf; x++) {
		for (y = max_nr; y >= 1; --y){
			printf("%s ", vector[x, y]);
		}
		printf("\n");
	}
}

$ echo '\
1 2 3 4 5 6\
2 3 4 5 6 1\
3 4 5 6 1 2\
4 5 6 1 2 3' >> multidimensional.data

$ awk -f multidimensional.awk multidimensional.data
4 3 2 1
5 4 3 2
6 5 4 3
1 6 5 4
2 1 6 5
3 2 1 6

```

### Functions



















































