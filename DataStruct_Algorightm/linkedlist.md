# 链表

链表是线型表结构，但不需要连续的内存空间，它通过“指针”将一组零散的内存块串联起来使用

![此处输入图片的描述][1]


  链表结构非常多，常见的链表结构：单链表、双向链表和循环链表
  
## 单链表

链表通过指针将一组零散的内存块串联起来，我们把内存块称为链表的结点

为了将所有结点串联起来，每个链表的结点除了存储数据之外，还需要记录链上的下一个结点的地址，把这个记录下一个结点地址的指针叫做后继指针 next

第一个结点（头结点）和最后一个结点（尾结点）比较特殊；头结点用来记录链表的基地址，尾结点的 next 指针指向一个空地址 NULL，表示这是链表的最后一个结点

![此处输入图片的描述][2]


因为链表本身内存不连续，所以在链表中插入和删除一个数据不需要搬运转移，所以操作数据非常快
  
![此处输入图片的描述][3]


但链表要随机访问一个元素就没有数组那么高效，它不能像数组一样通过首地址和下标来寻址对应的内存地址，而是需要根据指针一个结点一个结点的依次遍历，直到找到相应的结点
  
## 循环链表
跟单链表的区别在于尾结点不是指向 NULL，而是指向头结点

![此处输入图片的描述][4]


循环链表的优点是从链尾到链头比较方便
  
  
## 双向链表

双向链表支持两个方向，每个结点不止有一个后继指针 next 指向后面的结点，还有一个前驱指针 prev 指向前面的结点

![此处输入图片的描述][5]


如果存储同样多的数据，双向链表比单链表占用更多的内存空间，但支持双向遍历，带来了双向链表操作的灵活性；双向链表的删除、插入都比单向链表高效
  
## 双向循环链表
  
![此处输入图片的描述][6]


  
## 链表、数组比较
  
![此处输入图片的描述][7]


## 利用哨兵简化单链表操作
针对单链表的插入、删除操作，需要对插入第一个结点和删除最后一个结点的情况进行特殊处理，这样代码会
很繁琐、不简洁，而且容易因为考虑不全而出错

引入哨兵结点，在任何时候不管链表是不是空，head 指针都会一直指向哨兵结点；我们把这种有哨兵结点的链表叫带头链表，没有哨兵结点的链表叫做不带头链表

![此处输入图片的描述][8]


## 单链表有环的问题

[总结](https://www.cnblogs.com/hiddenfox/p/3408931.html)

```cpp
ListNode * fast = head, * slow = head;
while(fast && fast->next)
{
    fast = fast->next->next;
    slow = slow->next;
    if(fast == slow)
    {
        return true;  // 环
    }
    .
    .
    .
}
return false;  // 没有环

/*
1. 环的长度
方法一：快慢指针相遇后，让快指针不动，慢指针继续运动到再次相遇，走过的路程就是环的长度
方法二：环的长度等于第一次相遇时循环的次数

2. 环的开始节点
两个指针分别从第一个节点和快慢指针相遇的节点开始遍历，相遇处为环开始节点
*/
```

## 单链表翻转

```cpp
// 1.就地翻转
void reverseList(ListNode * head)
{
    if(!head)
    {
        return;
  	}

    ListNode dummy(-1), *pre = head, *cur = head->next;
    dummy.next = head;
    while (cur)
    {
        pre->next = cur->next;
        cur->next = dummy.next;
        dummy.next = cur;
        cur = pre->next;
    }
    head = dummy.next;
}
```
![](https://images0.cnblogs.com/blog2015/591083/201503/031550081605218.png)


```cpp
// 2. 新建链表，头结点插入
void reverseList(ListNode * head)
{
    if (!head)
    {
        return;
    }
    
    ListNode dummy(-1), *cur = head, *nex = head->next;
    while (cur)
    {
        cur->next = dummy.next;
        dummy.next = cur;
        cur = nex;
        if(nex) nex = nex->next;
    }
    head = dummy.next;
}
```
![](https://images0.cnblogs.com/blog2015/591083/201503/031603350829532.png)
![](https://images0.cnblogs.com/blog2015/591083/201503/031614382236340.png)

```cpp
// 3. 利用栈
void reverseList(ListNode * head)
{
	if (!head)
	{
		return;
	}
	
	ListNode dummy(-1), *cur = head;
	dummy.next = head;
	stack<ListNode *> s;
	while (cur)
	{
		s.push(cur);
		cur = cur->next;
	}
	cur = &dummy;
	while (!s.empty())
	{
		ListNode * temp = s.top();
		s.pop();
		cur->next = temp;
		cur = temp;
        cur->next = nullptr;
	}
    head = dummy.next;
}
```

```cpp
// 4. 递归
ListNode * head = nullptr;  // 全局变量，反转链表的头
void resverseList(ListNode * node)
{
    if(!node)
    {
        return;
    }

    if(node->next)
    {
        reverseList(node->next);
        node->next->next = node;
        node->next = nullptr;
    }
    else
    {
        head = node;
    }
}
```

### 两个链表相交

- 两个链表无环
```cpp
/*
1. 判断链表连接后是否有环
先遍历第一个链表到他的尾部，然后将尾部的 next 指针指向第二个链表头部第一个节点(尾部指针的 next 本来指向的是 null)，这样两个链表就合成了一个链表，判断原来的两个链表是否相交也就转变成了判断新的链表是否有环的问题了

从第二个链表的头部进行遍历，找到第一个和第一个链表重复的节点即是两个链表的交点；如果找到最后为 null，则说明两个单向链表不相交

这种方法不容易找到相交点
*/
```
![3piHRx.png](https://s2.ax1x.com/2020/02/16/3piHRx.png)

```cpp
/*
2. 从相同长度处开始遍历
两个链表如果相交的话，那么最后的一个节点一定是相同的，否则是不相交的

分别遍历到两个链表的尾部，然后判断他们是否相同，如果相同，则相交；否则不相交

假设第一个链表长度为 len1，第二个 len2，然后找出长度较长的，让长度较长的链表指针向后移动 len1 - len2，然后在开始遍历两个链表，判断节点是否相同即可
*/
```
![3pAzo4.png](https://s2.ax1x.com/2020/02/16/3pAzo4.png)

```cpp
/*
3. 使用 hash 表进行判断
4. 使用两个栈进行判断
*/
```

- 一个有环一个无环则不可能相交

- 两个都有环，相交于环外，两个链表共用一个环，从假装从环开始节点处断开，当做两个没有环的链表处理
![3pMMss.png](https://s2.ax1x.com/2020/02/16/3pMMss.png)

- 两个都有环，相交于入口处，两个链表共用一个环，则选取一条链找这条链的环开始节点即可
![3pMJiT.png](https://s2.ax1x.com/2020/02/16/3pMJiT.png)

- 两个都有环，相交于环内
```cpp
// 有两个相交点，各自的环入口节点即为各自第一次相交的节点
```
![3pMYJU.png](https://s2.ax1x.com/2020/02/16/3pMYJU.png)


  [1]: https://static001.geekbang.org/resource/image/d5/cd/d5d5bee4be28326ba3c28373808a62cd.jpg
  [2]: https://static001.geekbang.org/resource/image/b9/eb/b93e7ade9bb927baad1348d9a806ddeb.jpg
  [3]: https://static001.geekbang.org/resource/image/45/17/452e943788bdeea462d364389bd08a17.jpg
  [4]: https://static001.geekbang.org/resource/image/86/55/86cb7dc331ea958b0a108b911f38d155.jpg
  [5]: https://static001.geekbang.org/resource/image/cb/0b/cbc8ab20276e2f9312030c313a9ef70b.jpg
  [6]: https://static001.geekbang.org/resource/image/d1/91/d1665043b283ecdf79b157cfc9e5ed91.jpg
  [7]: https://static001.geekbang.org/resource/image/4f/68/4f63e92598ec2551069a0eef69db7168.jpg
  [8]: https://static001.geekbang.org/resource/image/7d/c7/7d22d9428bdbba96bfe388fe1e3368c7.jpg
  
  