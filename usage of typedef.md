# typedef用法

typedef为C语言的关键字，作用是为一种数据类型定义一个新名字。这里的数据类型包括内部数据类型（int,char等）和自定义的数据类型（struct等）。
在编程中使用typedef目的一般有两个，一个是给变量一个易记且意义明确的新名字，另一个是简化一些比较复杂的类型声明。
至于typedef有什么微妙之处，请你接着看下面对几个问题的具体阐述。



##  typedef & 结构的问题

------

#### 1、typedef的最简单使用

```c
typedef　long　byte_4;
```

给已知数据类型long起个新名字，叫byte_4

#### 2、 typedef与结构结合使用

```c
typedef　struct　tagMyStruct
{
int　iNum;
 
long　lLength;

}MyStruct;
```

这语句实际上完成两个操作：

1) 定义一个新的结构类型

```c
struct　tagMyStruct
{
int　iNum;
 
long　lLength;
 
};
```

分析：tagMyStruct称为“tag”，即“标签”，实际上是一个临时名字，struct关键字和tagMyStruct一起，构成了这个结构类型，不论是否有typedef，这个结构都存在。
我们可以用struct tagMyStruct varName来定义变量，但要注意，使用tagMyStruct varName来定义变量是不对的，因为struct 和tagMyStruct合在一起才能表示一个结构类型。

2) typedef为这个新的结构起了一个名字，叫MyStruct。



*一个错误的例子

```c
typedef　struct　tagNode
{
char*　pItem;
 
pNode*　pNext;
 
}pNode;
```

新结构建立的过程中遇到了pNext域的声明，类型是pNode，要知道pNode表示的是类型的新名字，那么在类型本身还没有建立完成的时候，这个类型的新名字也还不存在，也就是说这个时候编译器根本不认识pNode。

解决方法

1)

```c
typedef　struct　tagNode
{
char*　pItem;
 
struct　tagNode*　pNext;
 
}*pNode;
```

2)

```c
 typedef　struct　tagNode*　pNode;
 
struct　tagNode
{
char*　pItem;
 
pNode　pNext;//这边不用pNode* ，pNode 已经表示了struct tagNode*
}; 
```

注意：在这个例子中，你用typedef给一个还未完全声明的类型起新名字。C语言编译器支持这种做法。

### 3)、规范做法：

```c
struct　tagNode
{
char*　pItem;
 
struct　tagNode*　pNext;
};
 
typedef　struct　tagNode*　pNode;
```



#### 3.typedef & #define的问题

有下面两种定义pStr数据类型的方法，两者有什么不同？哪一种更好一点？

```c
typedef　char*　pStr;
 
#define　pStr　char*
```

通常讲，typedef要比#define要好，特别是在有指针的场合。请看例子：

```c
typedef　char*　pStr1;
 
#define　pStr2　char*　
 
pStr1　s1,s2;
 
pStr2　s3,s4;
```

在上述的变量定义中，s1、s2、s3都被定义为char *，而s4则定义成了char，不是我们所预期的指针变量，根本原因就在于#define只是简单的字符串替换而typedef则是为一个类型起新名字。

上例中#define语句必须写成 pStr2 s3, *s4; 这样才能正常执行。

以下是一个#define用法例子：

```c
#include <stdio.h>
#define f(x) x*x
int main(void)
{
    int a=6, b=2, c;
    c = f(a) / f(b);
    printf("%d\n", c);
    return 0;
}
```

以下程序的输出结果是: 36。

因为如此原因，在许多C语言编程规范中提到使用#define定义时，如果定义中包含表达式，必须使用括号，则上述定义应该如下定义才对：

```c
#definef(x)((x)*(x))
```

当然，如果你使用typedef就没有这样的问题。

#### 4. typedef & #define 的另一例

```c
typedef char *pStr;
char string[4]="abc";
const char *p1=string;
const pStr p2=string;
p1++;
p2++;
```

此程序运行会出错。

分析：是p2++出错了。这个问题再一次提醒我们：typedef和#define不同，它不是简单的文本替换。上述代码中const pStr p2并不等于const char * p2。const pStr p2和pStr const p2本质上没有区别，都是对变量进行只读限制，只不过此处变量p2的数据类型是我们自己定义的而不是系统固有类型而已。因此，const pStr p2的含义是：限定数据类型为char *的变量p2为只读，因此p2++错误。

#####define与typedef引申谈

1) #define宏定义有一个特别的长处：可以使用 #ifdef ,#ifndef等来进行逻辑判断，还可以使用#undef来取消定义。

 *****  #ifdef ,#ifndef,#undef

1.#ifdef ：当标识符已经被定义过(一般是用#define命令定义)，则对程序段1进行编译，否则编译程序段2。

```c
#ifdef 标识符
//程序段1
#else
//程序段2
#endif
```

2.#ifndef：只是第一行与第一种形式不同：将“ifdef”改为“ifndef”。它的作用是：若标识符未被定义则编译程序段1，否则编译程序段2。这种形式与第一种形式的作用相反。

```c
#ifndef 标识符
//程序段1
#else
//程序段2
#endif
```

3.#undef：#define定义预处理器标识符，将保持已定义状态且在作用域内，直到程序结束或者使用#undef 指令取消定义。

```c++
#include <iostream.h>
#include<string.h>
#define MAX 5
#undef MAX
void main()
{
char name[MAX]="abcd"; //只能用abcd,否则会提示说超出长度,原因是由于"\0"字符
cout<<"MAX = "<<MAX<<endl;
for(int i=0;i<MAX;i++)
cout<<name<<" "<<endl;
}
```

会得到“*未定义符号 'MAX'*”的错误。

2) typedef也有一个特别的长处：它符合范围规则，使用typedef定义的变量类型其作用范围限制在所定义的函数或者文件内（取决于此变量定义的位置），而宏定义则没有这种特性。

## 5.typedef & 复杂的变量声明

从变量名看起，先往右，再往左，碰到一个圆括号就调转阅读的方向；括号内分析完就跳出括号，还是按先右后左的顺序，如此循环，直到整个声明分析完。举例：

```c
int (*func)(int *p);
```

首 先找到变量名func，外面有一对圆括号，而且左边是一个* 号，这说明func是一个指针；然后跳出这个圆括号，先看右边，又遇到圆括号，这说明 ( * func)是一个函数，所以func是一个指向这类函数的指针，即函数指针，这类函数具有int*类型的形参，返回值类型是int。

```c
int (*func[5])(int *);
```

func 右边是一个[]运算符，说明func是具有5个元素的数组；func的左边有一个 * ，  说明func的元素是指针（注意这里的 * 不是修饰func，而是修饰 func[5]的，原因是[]运算符优先级比* 高，func先跟[]结合）。跳出这个括号，看右边，又遇到圆括号，说明func数组的元素是函数类型的指 针，它指向的函数具有int*类型的形参，返回值类型为int。

2个模式：

type (*)(....)函数指针

type (*)[]数组指针

