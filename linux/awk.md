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
```

















































