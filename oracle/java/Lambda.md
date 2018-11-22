# Lambda
使用lambda需要注意纯函数

### 纯函数
1. 函数的调用没有次数限制
2. 相同的输入产生相同的结果
3. 没有副作用-不会改变程序中的其他值

### 编写纯函数注意的原则:
1. 函数不要改变任何元素
2. 函数不依赖任何可能更改的元素

高阶函数-可以接收、创建或返回函数的函数或方法

lambda 是无状态的; 闭包是有状态的。将lambda表达式替换为闭包，是一种管理函数式程序中状态的好方法

```java
class Sample {
  public static Runnable create() {                   
    int value = 4;
    Runnable runnable = () -> System.out.println(value);
     
    System.out.println("exiting create");
    return runnable;
  } 
   
  public static void main(String[] args) { 
    Runnable runnable = create();
     
    System.out.println("In main");
    runnable.run();
  }
}
```
反编译后：
```java
//闭包字节码
private static void lambda$create$0(int);
    Code: 
       0: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
       3: iload_0
       4: invokevirtual #9                  // Method java/io/PrintStream.println:(I)V
       7: return
}
//create 字节码
0: iconst_4
1: istore_0
2: iload_0
3: invokedynamic #2,  0              // InvokeDynamic #0:run:(I)Ljava/lang/Runnable;
```
以上可知 `4` 存储在一个变量中，该变量被加载并传递到为闭包创建的函数。

函数编程中，通过一系列更小的模块化函数或运算来对复杂运算进行排序-函数组合；当一个数据集合流经一个函数组合时，就编程了一个集合管道




















