---
layout: post
title: Hack C 语言之零长数组
category: Clang
---

## 引子

《Redis 设计与实现》提到 Redis 有序集合的底层实现用的是跳跃表。跳跃表支持平均 $$O(\log N)$$ 、最坏 $$O(N)$$ 复杂度的节点查找。大部分情况下，跳跃表的效率可以和平衡二叉树相媲美，而且实现更为简单。Google 出品的 leveldb 同样使用了跳跃表。

之前没听过跳跃表的我，就想探个究竟，看看跳跃表（skiplist）是如何实现的。
跳跃表结构定义在 `redis.h` 中。

```C
typedef struct zskiplistNode {
    robj *obj;
    double score;
    struct zskiplistNode *backward;
    struct zskiplistLevel {
        struct zskiplistNode *forward;
        unsigned int span;
    } level[];    
} zskiplistNode;
```
`zskiplistNode` 结构的 `level` 字段的声明让我有点困惑，空方括号 `[]` 中没有指明大小，那么这个 `level` 是个指针吗？  
没有指明大小的数组在数组列表初始化和函数形参中见到过，但出现在结构体中，还是第一次见到。  
带着这个疑问，写代码测试了一下。

```C
struct A {
    int length;
    int body[];
};

int main() {
    struct A a;
    printf("sizeof(struct A)=%d\n", sizeof(a));
    return 0;
}
```
这个程序的输出是 `sizeof(struct A)=4`，并没有给 body 分配空间，那么 body 就不是一个指针。  
   


## 零长数组

变量只是机器内存地址的抽象，代码经过编译后，变量就被转换成了内存中的地址。访问变量，就是访问所在地址的内容。  
同样的，结构体的字段只是相对结构实例的地址偏移。添加代码如下：

```C
printf("       a at: 0x%x\n", &a);
printf("a.length at: 0x%x\n", &a.length);
printf("  a.body at: 0x%x\n", &a.body);
```
输出如下：

![结构体字段偏移]({{baseurl}}/assets/struct_field_offset.png)

`a.length` 地址就是结构体实例的 `a` 的地址，length 字段相对结构体的偏移是 0，body 字段相对结构体的偏移是 4。  

**这么做有什么用呢？** 

来看一个例子，要设计一个表示 email 的结构，有两种方法：  

其一

```C
struct email {
    time_t send_date;
    int flags;
    int length;
    char body[EMAIL_BODY_MAX];
};
```
嗯，为了让问题简单，故意省略了很多字段。这有一个问题，每个 email 实例都给 body 分配了 `EMAIL_BODY_MAX` 长度的空间，即使很多都用不到。  
body 长度不确定，很容易想到可以把 body 声明为指针，动态按需分配空间，就有了如下的代码：

```C
struct email {
    time_t send_date;
    int flags;
    int length;
    char *body;
};
```
这也有问题：  

- 给 body 申请的空间和 email 实例申请的空间不是连续的，用户释放 email 实例的空间时还要先释放 body 的内存。不利于内存释放。
- body 字段要占 4 字节内存（32位机器上）。

零长度数组就派上用场了。如下定义 email：

```C
struct email {
    time_t send_date;
    int flags;
    int length;
    char body[];  // same as char body[0]
};
```
使用方法如下：

```C
struct email *pmail = malloc(sizeof(struct email) + email_length);
pmail->length = email_length;
```
就可以达到按需分配空间，而且空间还是连续的，body 字段还不占用内存空间，真好。


## 灵活数组成员（flexible array member）

其实，这是 ISO C99 的特性，允许把结构体的最后一个成员定义为零长数组，来表示可变长度的 Object。
因此，零长数组也称为灵活数组。


参考链接：

1. [C语言结构体里的成员数组和指针](http://coolshell.cn/articles/11377.html) from 酷壳
2. [What is the purpose of a zero length array in a struct?](http://stackoverflow.com/questions/11733981/what-is-the-purpose-of-a-zero-length-array-in-a-struct) from stackoverflow
3. [Arrays of Length Zero](https://gcc.gnu.org/onlinedocs/gcc/Zero-Length.html) from GUN gcc

-EOF-