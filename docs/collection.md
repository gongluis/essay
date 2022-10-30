### collection集合  
[TOC]

#### 一、定义的理解
> java作为面向对象的语言，许多地方都是会对对象进行操作，那对于多个对象的操作，首相要进行存储，然后相应的操作，集合就是常用的存储方式，数组虽然也是个容器，但是只能存储基本数据类型，而且长度是固定的，不是很方便，集合只能存储对象，长度可以自增的，比较灵活。

#### 二、集合继承关系
```
graph TD
A[Iterator]-->B[Collection]
A-->C[Map]
B--无序不重复-->D[Set]
B--有序可重复-->E[List]
D--无序-->D1[HashSet]
D--排序-->D2[TreeSet]
E--查改优-->E1[ArrayList]
E--增删优-->E2[LinkedList]
C--无序-->C1[HashMap]
C--排序-->C2[TreeMap]
```
> vector和hashtable均被替代
```
vector与arralist区别，vector add方法同步，arraylist非同步，
vector扩容1倍，arraylist0.5倍，
vector构造方法是直接初始化10，arraylist第一次调用添加方法初始化容量为10，
vector是jdk1.0,arraylist是jdk1.2
vector是线程同步的效率低安全性高

增加或者删除较多使用linkedList，查找多使用arraylist.

hashtable

被concurrenthashmap替代
hashtable优点，同步，但是效率低。

```
#### 三、Collection
##### Set
###### 定义的理解
1. 为什么要使用这种容器，key和value相同，没有重复的value。无序，允许Null元素。
2. set是一个接口，实现类有：
    * hashset:hash算法存取对象，存取速度快
    * treeset: 能够对元素进行排序
###### 简单使用
```
Set<Integer> hashSet = new HashSet<Integer>();
 hashSet.add(3);
 hashSet.add(4);
 
 遍历
 for (Integer integer :hashSet){
            
   }
 Iterator<Integer> iterator =hashSet.iterator();
while (iterator.hasNext()){
     Integer integer=iterator.next();
   }
```
> 在放入自己定义的类实例时，需要重写hashcode和equals方法，用自己的关键字来重写，因为当使用HashSet时，hashCode()方法就会得到调用，判断已经存储在集合中的对象的hash code值是否与增加的对象的hash code值一致；如果不一致，直接加进去；如果一致，再进行equals方法的比较，equals方法如果返回true，表示对象已经加进去了，就不会再增加新的对象，否则加进去。


##### list
###### 定义的理解
1. 是一个继承collection的接口，实现类有：
    * ArrayList:查询快，增删慢，不同步，0.5增长
    * LinkedList:增删快
    * Vector: 都慢，被arraylist替代，线程安全
    * Stack: 继承于vector
##### map
###### 定义的理解
1. Map是个接口，实现类有：
    * HahMap: 非同步，无序数组+链表+红黑树
    * TreeMap:底层二叉树，不同步，可用于排序
    * LinkedHashMap:遍历时时是插入顺序，或者最近最少使用次序，速度稍慢，迭代更快
    * HashTable:
    * ConcurrentHashMap:并发效率更高用来替代hashtable
    ...
###### 使用方法

1. 方法  
    * 添加  
        * V put(K key, V value)可以相同的key，但是value会覆盖前一个
        * putAll(Map<? extend K, ? extends V> m)
    * 删除
        * remove() 删除关联对象，指定key对象
        * clear() 清空集合对象
    * 获取  
        * value get(key) 可以用于判断是否存在的情况，指定的key不存在时返回的是null
    * 判断
        * boolean isEmpty()长度为0返回true,否则false
        * boolean containsKey(Object key) 判断集合中是否包含key
        * boolean containsValue(Object value),是否包含指定value
    * 长度
        * int size()

#### 四、常见问题

1. 为什么arraylist插入效率低于linklist?

   可以看作是一个数组的数据结构为啥插入效率低于链表数据结构的问题，从arraylist的add方法源码可以知道，插入一个元素到索引为i的位置，是要把从i开始往后的所有元素做一个arrayCopy，总体向后移动一位来做到的，删除元素是同样总体做arrayCopy往前移动一个位置。

   链表结构的每一个元素都是一个node，包含next prev data三个属性，插入一个元素，只需要修改上一个元素的next属性和下一个元素的prev属性，只要修改引用就可以了，所以速度是很快的。

   

2. 为什么arraylist查找效率高于linklist?

   arraylist数组数据结构直接拿着下标查询即可，连标需要做轮询查找。

3. 如果一个数组要反复的插入删除怎么优化降低时间复杂度？

   时间复杂主要浪费在频繁的插入需要频繁的进行数组的arrayCopy，这个时候我们可以借助jvm的垃圾回收算法中的markswap算法的思路，先对垃圾进行标记，然后统一进行清除，这里可以先对删除的索引做个标记，比如标记为null，在内存不足的时候进行统一清理。以空间换时间。
