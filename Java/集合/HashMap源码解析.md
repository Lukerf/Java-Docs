结构是数组+链表+红黑树

### put操作

通过key，找到对应的Hash槽，如果对应Hash槽没有初始化，则初始化。然后判断是红黑树还是普通链表的节点（Node还是TreeNode），将节点插入链表或者红黑树。

链表什么时候会转化为红黑树：链表元素大于**8**，且数组长度大于等于**64**的时候，才会将链表转化为红黑树

### 扩容机制

当数组的占用数量达到 **数组大小*负载因子** 时，数组大小会扩展为原来的两倍



### 1.7和1.8的区别

1.7结构是用链表解决hash冲突，1.8以后增加了红黑树来提升链表的查询速度

1.7在往链表中插入数据的时候使用的是头插法，1.8是尾插入法。可以避免在多线程使用HashMap时，出现环形链表死循环的问题