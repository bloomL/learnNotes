#### 编译解释型语言

> 编译

**编译**为.class文件

> 解释

通过**解释器**解释为操作系统运用

#### 类型转换

低 -------------------------------->高

byte、short、char --> int --> long --> float --> double

1. 强制类型转换
2. 自动类型转换

#### 变量作用域

1. 类变量

   1.1  static修饰  类变量

2. 实例变量

   2.1 属于对象，默认值

3. 局部变量

   3.1 方法中

#### 运算符

取余叫做**取模** 

位运算

```bash
&	与 ：同时为1，才为1
|	或 ：同时为0，才为0
^	异或 ：相同为0，不同为1
~	取反 ： 0变1，1变0
<< 左移 *2
>> 右移 /2
```

```java
A = 0011 1100
B = 0000 1101

A&B = 0000 1100
A|B = 0011 1101
A^B = 0011 0001
~B = 1111 0010
```



case**穿透**

```java
char grade = 'A';
switch(grade){
    case 'A':
        System.out.println("优秀");
        break;
    case 'B':
        System.out.println("优秀");
    case 'C':
        System.out.println("优秀");
    case 'D':
        System.out.println("优秀"); 
    default:
        System.out.println("优秀");   
}
```







