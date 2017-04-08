# 3.3链表

## 1.定义

链表是一组数据项的集合，其中每个数项都是一个节点的一部分，每个节点都包含指向下一个节点的链接。

我们用指针表示链接，用结构体表示节点：

```c
typedef struct node *link;
struct node
{
  Item item;
  link next;
};
```



### 约瑟夫问题

假设N个人决定选出一个领导人，方法如下：

​     所有人排成一个圆圈，按顺序数数，每隔第M个人出局，此时其他两边的人重新形成圆圈。问题是找出哪一个会是最后剩下的人。。所选出的领导人是一个关于N和M的函数，我们称之为约瑟夫问题（Josephus function)

解决的思想是：

1构造一个循环链表表示排成圆圈的人，每人个的链接指向圆圈内在他左边的人，整数i表示第i个人。

2.在1号构造一个循环链表之后，再把2~N号插入到1号节点之后，得到1~N号的圆环，并使x指向N。

3.从1号开始，跳过M-1个节点，把第M-1节点指向M+1节点

4继续3，直到剩下最后一个节点。

```c
#include<stdio.h>
#include<stdlib.h>
typedef struct node *link;
struct node
{
    int item;
    link next;
};
int main ()
{
    int i ,N,M;
    scanf("%d",&N);
    scanf("%d",&M);
    link t=malloc(sizeof *t),x=t;
    t->item=1;t->next=t;//构造循环链表
    for(i=2;i<=N;i++)
    {
        x=(x->next=malloc(sizeof *x));
        x->item=i;
        x->next=t;
    }//读入剩下的人
    while(x!=x->next)
    {
        for(i=1;i<M;i++)x=x->next;
        x->next=x->next->next;
        N--;//不断删除
    }
    printf("%d",x->item);
    return 0 ;

}
```

## 2.基本操作

链表更有限制性的定义：

链表是一个空表，或是一个指向节点的链接，且这个节点包含一个元素和一个指向链表的链接.

(1)链表求逆问题：定义函数把链表逆序，并返回一个指向尾节点的指针，这个指针指向倒数第二个节点。而原链表的首节点设为NULL。

分析：我们需要保持指向链表中已经处理完毕部分的指针r和指向没有处理完毕的指针y。先把y的后节点保存在t中，然后使y的链接指向r，再使r移到y，y移到t.

```c
link reverse(link x)
  {
    link t ,y = x,r=NULL;
    t=y->next;
    y->next=r;
    r=y;
    y=t;
  return t ;
  }
```

(2)链表插入排序问题：产生0~999之间的N个随机数，构建每个节点代表一个数的链表，重新排列这些节点，使遍历链表时这些数按照顺序输出现。

实现排序步骤：

1.保存两个链表，一个输入（无序）链表，一个输出（有序）链表。每个链表使用一个指向链表首节点的头节点。

2.在循环的每次迭代过程中，从输入链表中取出一个节点，并把它插入到输出链表的适当位置。

```c
#include<stdio.h>//有不可名状的错误！！！！！！！！！！！！！！！！！！！！！！！来来来找找bug
#include<stdlib.h>
typedef struct node *link;
struct node
{
    int item;
    link next;
};
int main ()
{
    struct node heada,headb;
    link t,u,x,a=&heada,b=&headb;
    int i ,N;
    scanf("%d",&N);
    for(i=0,t=a;i<N;i++)
    {
        t->next=malloc(sizeof*t);
        t=t->next;t->next=NULL;
        t->item=rand()%1000;
    }
    b->next=NULL;
    for(t=a->next;t!=NULL;t=u)
    {
        u=t->next;
        for(x=b;b->next!=NULL;x=x->next)
            if(x->next->item>t->item) break;
        t->next=x->next;
        x->next=t;
    }
    for(t=&headb;t->next!=NULL;t=t->next)
    {
        printf("%d",t->item);
    }
    return 0;
}
```

### 链表中头和尾节点/指针的常规用法

#### (1)循环、永远非空

头插入：head->next=head;????

在x节点后插入t节点：t->next = x->next; x->next = t;

删除x后的节点：x->next=x->next->next;

遍历循环：t=head;

do

{...t=t->next;}while(t!=head

测试是否只有一个元素：if(head->next==head)

#### (2)头指针、尾节点为空

初始化 head=NULL;

在x节点后插入t节点：if(x==NULL){head=t;head->next=NULL;}

​                                        else{t->next=x->next;x->next=t}

删除x后的节点：t=x->next;x->next=t->next;free(t);

遍历循环：for(t=head;t!=NULL;t=t->next)

测试是否只有一个元素：if(head==NULL)

#### (3)有哑节点、尾节点为空

？???？？？？？初始化：head=malloc(sizeof *head)

​			head->next=NULL;

在x节点后插入t节点：t->next=x->next;x->next=t;

删除x后的节点：t=x->next;x->next=t->next

遍历循环：for(t=head->next;t!=NULL;t=t->next)

测试链表是否为空：if(head->next==NULL)

#### (4)有哑节点/尾节点

???

## 3.链表处理接口实现

链表的一些常用接口

```c
#include<stdio.h>
#include<stdlib.h>
typedef struct node *link;
struct node{itemType item;link next;};
typedef link Node
void initNodes(int);
link newNode(int);
void freeNode(link);
void insertNext(link,link);
link deleteNext(link);
link Next(link);
int Item(link);

void initNodes(int N)
  {
    int i ;
    freelist = malloc((N+1)*(sizeof *freelist));
    for(i=0;i<=N;i++)
       freelist[i].next = &freelist[i+1];
    freelist[N].next = NULL;
  }
link nexNode(int i )
  {
    link x = deleteNext(freelist);
    x->next=i;x->next = x;
    return x;
  }
void freeNode(link x)
  {
    insertNext(freelist,x);
  }
void insertNext(link x;link t)
  {
    t->next = x->next;
    x->next = t ;    
  }
void deleteNext(link x)
  {
    link t = x->next;
    x->next=t->next;
    return t;
  }
link Next(link x)
  {
    return x->next;
  }
int Item(link x)
  {
    return x->item;
  }
```





