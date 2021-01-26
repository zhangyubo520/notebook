# HashMap

### 1、概念

```
是一种容器，jdk1.7数据结构是数组+链表，链表的节点存储的是entry对象，entry对象存储的是hash，key,value,next
jdk1.8数据结构是数组+链表+红黑树，链表节点存储的是Node对象

类定义：public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable
```

### 2、集合回顾

| 接口   | 实现         | 历史集合类   |
| ------ | ------------ | ------------ |
| `Set`  | `HashSet`    |              |
|        | `TreeSet`    |              |
| `List` | `ArrayList`  | `Vector`     |
|        | `LinkedList` | `Stack`      |
| `Map`  | `HashMap`    | `Hashtable`  |
|        | `TreeMap`    | `Properties` |

### 3、1.7中hashMap的put原理

```
1、根据HashMap内部hash算法重新计算key的hash值

2、通过计算出来的hash值调用indexFor方法计算当前对象应该在底层数组的几号位置

3、判断size(容器中已有entry的数量)是否达到阈值，如果没有，继续下一步，如果达到阈值，先将数组扩容至原来的2倍

4、将当前对象的hash，key,value封装成一个entry，去数组中查找当前位置上有没有元素，如果没有，直接把entry放入数组

4、如果数组当前位置上已经有链表，遍历链表，如果链表中某个节点的key与当前key进行equals比较后结果为true，将原节点上的value返回，用新的value替换原来的value，如果遍历完链表，没有找到某个节点的key与当前key的equals比较后结果为true，就把封装好的entry放入链表的第一个位置
```

### 4、1.8中hashMap的put原理

```
源码：
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;//声明一个Node数组tab，一个Node元素p,数组长度为n,i为数组索引
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;//添加第一个元素时初始化Node数组长度为16
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);//如果使用hash值求出当前对象在数组的索引位置上为空(计算方法为(n - 1) & hash)
        else {//如果当前数组索引位置上有链表(已经有元素，非空)
            Node<K,V> e; K k;//声明一个Node元素p,K为泛型，和要插入的key同类型
            if (p.hash == hash &&//如果新添加的元素和原位置上元素的hash值相等
                    ((k = p.key) == key || (key != null && key.equals(k))))//并且key值使用equals方法返回true
                e = p;//将旧的Node元素赋值给e
            else if (p instanceof TreeNode)//如果原数组位置上为红黑树
                e = ((HashMap.TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {//遍历链表，binCount是链表元素个数
                    if ((e = p.next) == null) {//插入到链表末尾
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st -1代表第一次插入
                            treeifyBin(tab, hash);//binCount=0时代表插入前有一个元素，等于7时插入前时有8个元素
                        break;
                    }
                    if (e.hash == hash &&
                            ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

### 5、扩容机制

```
懒扩容：
1、当put的时候才会判断是否需要扩容
2、数组扩容为原来的2倍
3、扩容后会将原来数组中的元素重新计算数组位置，并放入新数组中
4、使用hash & (length - 1)的原因是与运算比取模运算性能高，为此，hashmap初始化的时候，数组长度 length必须是2的整次幂（如果手动传参数组长度为奇数n，hashMap会自动转换长度为距离n最近的2的整次幂数）只有这样，与运算的结果才能和取模运算的结果一样
5、底层hash算法：(key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16)
6、阈值 = 容量 * 负载因子 = 16 * 0.75 = 12
7、如果插入节点后节点数超过阈值，则调用resize方法进行扩容

扩容的两个条件：
1、存放新值时，已有元素大于阈值
2、存放新值时，发生了hash碰撞（当前key计算的hash值求出的数组索引位置上已经有元素了）

推论：
1、最多存储16个元素时才会发生第一次扩容（因为每次存储都没有hash碰撞，第17个元素必然发生碰撞，并且已经超过阈值12）

2、最少11个元素会树形化(因为在同一个索引位置插入8个元素时，不会扩容，也不会树形化，在同一个位置插入第9个元素时，第一次扩容为32，在同一个位置插入第10个元素时，扩容为64，在同一个位置插入第11个元素时，树形化)

```

### 6、HashMap和HashTable

```
1、两者存储结构和解决冲突的方式相同

2、hashtable中默认数组大小是11，扩容为原来的2倍+1，不要求底层数组的容量为2的整数次幂，hashmap默认数组大小是16，扩容为2倍，要求底层数组的容量为2的整数次幂，如果传入的参数不是2的整数次幂，系统会自动调整为2的整数次幂（比如传入的是5，系统自动调整为8），源码如下：
/**
     * Returns a power of two size for the given target capacity.
     */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }

3、hashtable的key和value不允许有null，hashmap允许有一个key为null，多个value为null

4、hashtable中计算hash值是使用key的hashCode(),hashmap中计算hash值是

5、hashtable中计算hash值对应的数组位置时使用的是取模运算，hashmap计算hash值对应的数组位置使用的是取模运算

6、hashtable是线程安全的，hashmap不是
```

### 7、负载因子为啥是0.75

```
1、负载因子过大，map内数组的利用率很高，但是很容易形成entry链，查询效率降低
2、负载因子过小，map内数组的利用率较低，查询效率很高，但是浪费内存
```

### 8、8作为链表转红黑树的阈值

```
根据泊松分布，在负载因子默认为0.75的时候，单个hash槽内元素个数为8的概率约6000万分之一，所以将7作为一个分水岭，等于7的时候不转换，大于等于8的时候才进行转换，小于等于6的时候就化为链表。
```

### 9、hashmap和concurrenthash

```

```



