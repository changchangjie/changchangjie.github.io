---
title: Java集合-Map之HashMap
date: 2018-10-12 17:00:00
categories: 
- java集合
---
JDK1.8 之前 HashMap 由 数组+链表 组成的，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）.JDK1.8 以后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）时，将链表转化为红黑树，以减少搜索时间
<!--more-->

### 1.8之前底层数据结构
JDK1.8 之前 HashMap 底层是 数组和链表 结合在一起使用。 key经过扰动函数处理过后得到 hash 值，然后通过 (n - 1) & hash 判断当前元素存放的位置（这里的 n 指的时数组的长度），如果当前位置存在元素的话，就判断该元素与要存入的元素的 hash 值以及 key 是否相同，如果相同的话，直接覆盖，不相同就通过拉链法解决冲突。

所谓扰动函数指的就是 HashMap 的 hash 方法。使用扰动函数之后可以减少碰撞

1.8之前的扰动函数
```java
static int hash(int var0) {
    var0 ^= var0 >>> 20 ^ var0 >>> 12;
    return var0 ^ var0 >>> 7 ^ var0 >>> 4;
}
```
1.8扰动函数
```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
相比于 JDK1.8 的 hash 方法 ，JDK 1.7 的 hash 方法的性能会稍差一点点，因为毕竟扰动了 4 次

所谓 “拉链法” 就是：将链表和数组相结合。也就是说创建一个链表数组，数组中每一格就是一个链表。若遇到哈希冲突，则将冲突的值加到链表中即可
![](https://camo.githubusercontent.com/eec1c575aa5ff57906dd9c9130ec7a82e212c96a/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f332f32302f313632343064626363333033643837323f773d33343826683d34323726663d706e6726733d3130393931)

### 1.8之后
相比于之前的版本，jdk1.8在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间

### 属性
```java
// 默认的初始容量是16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;   
// 最大容量，超过该容量就不再扩容了
static final int MAXIMUM_CAPACITY = 1 << 30; 
// 默认的填充因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;
// 当桶(bucket)上的结点数大于这个值时会转成红黑树
static final int TREEIFY_THRESHOLD = 8; 
// 当桶(bucket)上的结点数小于这个值时树转链表
static final int UNTREEIFY_THRESHOLD = 6;
// 桶中结构转化为红黑树对应的table的最小大小
static final int MIN_TREEIFY_CAPACITY = 64;
// 存储元素的数组，总是2的幂次倍
transient Node<k,v>[] table; 
// 存放具体元素的集
transient Set<map.entry<k,v>> entrySet;
// 存放元素的个数，注意这个不等于数组的长度。
transient int size;
// 每次扩容和更改map结构的计数器
transient int modCount;   
// 临界值 当实际大小超过临界值(容量*填充因子)时，会进行扩容
int threshold;
// 填充因子
final float loadFactor;
```
### loadFactor填充因子
loadFactor填充因子是控制数组存放数据的疏密程度
loadFactor越趋近于1，导致临界值越大，越难扩容，数据越密，链表越长，查找效率越低
loadFactor越趋近于0，导致临界值越小，越易扩容，数据越散，数组越大，增删效率越低
默认值为0.75f是官方给出的一个比较好的临界值。 　

### threshold临界值
当实际大小超过临界值(容量 * 填充因子)时，会进行扩容
默认临界值是16*0.75=12，即超过12个元素时会扩容

### 扩容
因为HashMap为了节省创建出的对象的内存占用，一开始只默认分配16个，超过临界值就会进行扩容，扩大成2倍，容量达到最大值则不再扩容
进行扩容，会伴随着一次重新hash分配，并且会遍历hash表中所有的元素，是非常耗时的。在编写程序中，要尽量避免resize

### 寻址函数h&(length - 1)
寻址函数相当于对数组长度进行取模，而且速度比直接取模快得多，除了取模运算外还有一个非常重要的责任：均匀分布table数据和充分利用空间

### 数组长度为什么总是2的n次方
&运算符：如果相对应位都是1，则结果为1，否则为0
在构造函数中存在：capacity <<= 1;这样做总是能够保证HashMap的底层数组长度为2的n次方
length为2的幂次方，即一定是偶数，偶数减1，即是奇数，这样保证了（length-1）在二进制中最低位是1，而&运算结果的最低位是1还是0就完全取决于hash值二进制的最低位。如果length为奇数，则length-1则为偶数，则length-1二进制的最低位恒为0，则&位运算的结果最低位恒为0，即恒为偶数。这样table数组就只可能在偶数下标的位置存储了数据，浪费了所有奇数下标的位置，这样也更容易产生hash冲突

### 1.6源码分析
1. 构造方法
```java
//无参
public HashMap() {
    this.entrySet = null;
    this.loadFactor = 0.75F;
    this.threshold = 12;
    this.table = new HashMap.Entry[16];//数组长度
    this.init();
}
//指定数组长度
public HashMap(int var1) {
	this(var1, 0.75F);
}
//初始化容量和填充因子
public HashMap(int var1, float var2) {
    this.entrySet = null;
    if (var1 < 0) {
    	throw new IllegalArgumentException("Illegal initial capacity: " + var1);
    } else {
        if (var1 > 1073741824) {
            var1 = 1073741824;
        }

        if (var2 > 0.0F && !Float.isNaN(var2)) {
            int var3;
            //保证HashMap的底层数组长度为2的n次方
            for(var3 = 1; var3 < var1; var3 <<= 1) {
                ;
            }

            this.loadFactor = var2;
            this.threshold = (int)((float)var3 * var2);
            ////初始化table数组，这是HashMap真实的存储容器
            this.table = new HashMap.Entry[var3];
            //该方法为空实现，主要是给子类去实现
            this.init();
        } else {
        	throw new IllegalArgumentException("Illegal load factor: " + var2);
        }
    }
}
```
![HashMap结构](https://upload-images.jianshu.io/upload_images/4843132-05b3a55bd2686dd3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600/format/webp)
2. put

思路：用key的hashcode经过扰动函数和寻址函数找到数组下标，再用头插法插到所在entry的最前面
```java
    public V put(K var1, V var2) {
        if (var1 == null) {
            return this.putForNullKey(var2);
        } else {
        	//根据key的hashcode获取扰动值
            int var3 = hash(var1.hashCode());
            //寻址，即对数组长度进行取模,算出所在数组下标
            int var4 = indexFor(var3, this.table.length);
			//遍历数组元素的链表
            for(HashMap.Entry var5 = this.table[var4]; var5 != null; var5 = var5.next) {
            	//不同变量的hash值可能会相同
                if (var5.hash == var3) {
                    Object var6 = var5.key;
                    //key值存在的话替换value值，返回旧值
                    if (var5.key == var1 || var1.equals(var6)) {
                        Object var7 = var5.value;
                        var5.value = var2;
                        var5.recordAccess(this);
                        return var7;
                    }
                }
            }

            ++this.modCount;
            this.addEntry(var3, var1, var2, var4);
            return null;
        }
    }
    void addEntry(int var1, K var2, V var3, int var4) {
        HashMap.Entry var5 = this.table[var4];//数组对应的entry
        //构建新的entry,将原entry作为新entry的next元素,即头插法,插到链表的最前面
        this.table[var4] = new HashMap.Entry(var1, var2, var3, var5);
        if (this.size++ >= this.threshold) {
        	//扩容
            this.resize(2 * this.table.length);
        }
    }
    void resize(int var1) {
        HashMap.Entry[] var2 = this.table;
        int var3 = var2.length;
        if (var3 == 1073741824) {
            this.threshold = 2147483647;
        } else {
        	//创建一个新的table，长度为之前的两倍
            HashMap.Entry[] var4 = new HashMap.Entry[var1];//
            this.transfer(var4);
            this.table = var4;
            //重新计算临界值
            this.threshold = (int)((float)var1 * this.loadFactor);
        }
    }
    //通过遍历的方式，将老table的数据，重新寻址并存储到新table的适当位置
    void transfer(HashMap.Entry[] var1) {
        HashMap.Entry[] var2 = this.table;
        int var3 = var1.length;

        for(int var4 = 0; var4 < var2.length; ++var4) {
            HashMap.Entry var5 = var2[var4];
            if (var5 != null) {
                var2[var4] = null;

                HashMap.Entry var6;
                do {
                    var6 = var5.next;
                    //重新寻址
                    int var7 = indexFor(var5.hash, var3);
                    var5.next = var1[var7];
                    var1[var7] = var5;
                    var5 = var6;
                } while(var6 != null);
            }
        }
    }
```
3. get(Object key)

思路：用key的hashcode经过扰动函数和寻址函数找到数组下标，再遍历单向链表
```java
public V get(Object var1) {
        if (var1 == null) {
            return this.getForNullKey();
        } else {
            int var2 = hash(var1.hashCode());
            for(HashMap.Entry var3 = this.table[indexFor(var2, this.table.length)]; var3 != null; var3 = var3.next) {
                if (var3.hash == var2) {
                    Object var4 = var3.key;
                    if (var3.key == var1 || var1.equals(var4)) {
                        return var3.value;
                    }
                }
            }
            return null;
        }
    }
```
4. remove(Object var1)
思路：同样用key的hashcode经过扰动函数和寻址函数找到数组下标，再操作单向链表
```java
	public V remove(Object var1) {
        HashMap.Entry var2 = this.removeEntryForKey(var1);
        return var2 == null ? null : var2.value;
    }

    final HashMap.Entry<K, V> removeEntryForKey(Object var1) {
        int var2 = var1 == null ? 0 : hash(var1.hashCode());
        int var3 = indexFor(var2, this.table.length);
        HashMap.Entry var4 = this.table[var3];

        HashMap.Entry var5;
        HashMap.Entry var6;
        for(var5 = var4; var5 != null; var5 = var6) {
            var6 = var5.next;
            if (var5.hash == var2) {
                Object var7 = var5.key;
                if (var5.key == var1 || var1 != null && var1.equals(var7)) {
                    ++this.modCount;
                    --this.size;
                    if (var4 == var5) {
                        this.table[var3] = var6;
                    } else {
                        var4.next = var6;
                    }

                    var5.recordRemoval(this);
                    return var5;
                }
            }

            var4 = var5;
        }

        return var5;
    }
```
5. entrySet()
EntrySet重写了iterator方法
```java
public java.util.Map.Entry<K, V> next() {
	return this.nextEntry();
}
final HashMap.Entry<K, V> nextEntry() {
    if (HashMap.this.modCount != this.expectedModCount) {
    	throw new ConcurrentModificationException();
    } else {
    	HashMap.Entry var1 = this.next;
        if (var1 == null) {
        	throw new NoSuchElementException();
        } else {
            // 如果遍历到table单向链表的最后一个元素时
    		if ((this.next = var1.next) == null) {
    			HashMap.Entry[] var2 = HashMap.this.table;
				//继续往下寻找table上有元素的下标
                while(this.index < var2.length && (this.next = var2[this.index++]) == null) {
                    ;
                }
            }
            this.current = var1;
            return var1;
    	}
    }
}
```
可知，HashMap的遍历，是依次遍历table上每一条单向链表，这不是插入的顺序，所以说：HashMap是无序的

### key为null的处理

1. key为null的put
其实就是放在table[0]的链表里
```java
	private V putForNullKey(V var1) {
        for(HashMap.Entry var2 = this.table[0]; var2 != null; var2 = var2.next) {
            if (var2.key == null) {
                Object var3 = var2.value;
                var2.value = var1;
                var2.recordAccess(this);
                return var3;
            }
        }

        ++this.modCount;
        //插入到table[0]上单向链表的头部
        this.addEntry(0, (Object)null, var1, 0);
        return null;
    }

```
2. key为null的get
其实就是遍历table[0]的单向链表
```java
	private V getForNullKey() {
        for(HashMap.Entry var1 = this.table[0]; var1 != null; var1 = var1.next) {
            if (var1.key == null) {
                return var1.value;
            }
        }

        return null;
    }
```
所以，在HashMap中，不允许key重复，而key为null的情况，只允许一个key为null的Entry，并且存储在table[0]的单向链表上

### 总结
1. HashMap是基于哈希表实现的，用Entry[]来存储数据，而Entry中封装了key、value、hash以及Entry类型的next
2. HashMap存储数据是无序的
3. hash冲突是通过拉链法解决的
4. HashMap的容量永远为2的幂次方，有利于哈希表的散列
5. HashMap不支持存储多个相同的key，且只保存一个key为null的值，多个会覆盖
6. put过程，是先通过key算出hash，然后用hash算出应该存储在table中的index，然后遍历table[index]，看是否有相同的key存在，存在，则更新value；不存在则插入到table[index]单向链表的表头，时间复杂度为O(n)
7. get过程，通过key算出hash，然后用hash算出应该存储在table中的index，然后遍历table[index]，然后比对key，找到相同的key，则取出其value，时间复杂度为O(n)
8. HashMap是线程不安全的，如果有线程安全需求，推荐使用ConcurrentHashMap

### 参考链接
> [图解HashMap原理](https://www.jianshu.com/p/dde9b12343c1)  













