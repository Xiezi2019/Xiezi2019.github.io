2020-7-21-java集合-List

- ArrayList:基于动态数组实现，支持随机访问；
- Vector:线程安全，syschronized同步
- LinkedList:双向链表实现，删除和插入时间复杂度O(1)，顺序查找元素O(n),还可用作栈和队列；

###### ArrayList

- 基于动态数组实现，默认大小为10；

- 扩容：添加元素时用ensureCapacityInternal()方法保证容量足够，不够使grow()扩容，1.5倍；

- 扩容是将整个原数组复制到新的数组中，代价高，最好在一开始设置合适容量；

- 删除元素：调用System.arrayCopy()，将index+1位置上的元素复制到index上，O(n);

- 序列化:数组实现，动态扩容，不能保证所有元素被使用，无需全部序列化；实现了WriteObject()和readObject()保证只又被元素填充的部分被序列化；

- modCount变量记录数组结构变化，元素的添加删除操作或者调整数组内部大小；

- 删除元素： 不能用for ecah,实际使用的是iterator(),并发错误；iterator.remove()(modCount重新赋给expectModCount)、for循环正向遍历（改变下标）、for循环倒序遍历（不变下标） modCount和expectmoCount;

  ###### 

###### voctor

- syscchronized,同步，访问更慢
- 扩容2倍

###### 替代方案

- copyOnWriteArrayList:读写分离，写在复制数组上进行，读在原数组中进行。写操作加锁，写完把原数组复制到新的复制数组

  缺陷：内存占用，实时性无法保证；

- Collections.syschronizedList(List list)包装