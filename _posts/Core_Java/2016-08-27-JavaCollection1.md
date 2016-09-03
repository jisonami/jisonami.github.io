---
layout: post
title:  "Java集合框架（一）--集合API与数据结构的关系"
date:   2016-08-27 15:14:54
categories: Core_Java
tags: Java Collection 数据结构
---

* content
{:toc}

Java的集合框架是对常用数据结构的实现，Java程序员每天都会用到集合框架，但是你对它真的了解吗？

我依旧记得我第一份工作中我的同事问我为什么在HashMap中取得数据没有按照存数据的顺序拿出来，而是乱序的，怎么对HashMap进行排序啊？

如果他对集合框架足够了解的话，就会知道使用LinkedHashMap可以维持存入数据的顺序，使用TreeMap存入数据时就已经对TreeMap里的数据排好序了。   




### Java集合框架类图   

![Java集合类图](/images/Core_Java/JavaCollection1/JavaCollectionClassDiagram.png)   

把集合框架中继承关系的抽象类去掉了，紫色的是接口，白色的是具体实现类。

任何时候都应该优先考虑使用接口而不是实现类编程，除非接口中不包含需要的方法。 

Iterator代表可以直接迭代的集合，Map并没有提供直接迭代的操作，所以Map被设计成与Iterator没有继承关系。 

### 按照继承关系可以分为两大类
 
1、集合元素为对象的Collection集合，
 
2、集合元素为键值对的Map集合。 

### 按照实现数据结构的类型来分 

1、Queue接口代表的是队列，其子接口Deque代表的是双端队列，实际上Deque既包含了队列的方法也包含了栈的方法。 

2、List接口代表的是有序列表 

3、Set接口代表的是无序列表 

4、Map接口代表是关联数组 

### 下面接着说具体的实现类：
 
#### PriorityQueue

PriorityQueue是对优先队列的实现

也就是，插入PriorityQueue的元素默认会按照自然顺序从小到大排序，例如: 

```java
    Queue<Integer> pq = new PriorityQueue<Integer>();  
    pq.offer(4);  
    pq.offer(3);  
    pq.offer(5);  
    System.out.println("遍历PriorityQueue：");  
    for (Integer e : pq) {  
        System.out.println(e);  
    }  
    System.out.println("从PriorityQueue中取数据：");  
    while(!pq.isEmpty()){  
        System.out.println(pq.poll());  
    }     
```

offer()方法是往队列尾部插入一个元素，与add()方法是一样的，实际上add()方法的实现就是直接调用offer()方法而已。

使用Poll()方法每次取数据都会按照队列的方式取集合中第一个数据，并且把取了的元素删掉。

上面这段代码的执行结果如下： 

![优先队列PriorityQueue](/images/Core_Java/JavaCollection1/PriorityQueue.png)


当然了，这个插入元素时的排序规则我们也是可以自定义的，

只要在构造PriorityQueue对象是传入Comparator比较器接口的实现对象就可以了，或者保证插入PriorityQueue的元素均实现了Comparable接口。

因为PriorityQueue在实际编程中用的似乎不多，比较器的用法请参考HashSet与HashMap部分。 

#### ArrayDeque

ArrayDeque是对双端队列的数组实现 

前面说了，双端队列既可以当做队列使用也可以当做栈来使用，看代码实现： 

```java
    Deque<Integer> ad = new ArrayDeque<Integer>();  
    System.out.println("将ArrayDeque当做队列使用：(先进先出)");  
    ad.offer(4);  
    ad.offer(3);  
    ad.offer(5);  
    while(!ad.isEmpty()){  
        System.out.println(ad.poll());  
    }  
    System.out.println("将ArrayDeque当做栈使用：（后进先出）");  
    ad.push(5);  
    ad.push(3);  
    ad.push(4);  
    while(!ad.isEmpty()){  
        System.out.println(ad.pop());  
    }  
```

上面这段代码执行结果如下： 

![双端队列ArrayDeque](/images/Core_Java/JavaCollection1/ArrayDeque.png) 


#### ArrayList	

ArrayList应该是我们用的最多的集合类了，对应数据结构里的顺序表的概念，一组地址连续的存储单元构成的集合。

基于数据实现，但是多了自动扩容的功能，其实相当于一个变长数组。

不过Java中数组创建时长度就定了，所谓的自动扩容其实是新建了一个新数组，然后将原来数组的值复制到新数组里面。

你可能会认为ArrayList的自动扩容效率很低，其实ArrayList赋值数组的方法使用的是Java虚拟机实现的本地方法，直接操作内存空间，并不是一个个元素遍历来复制的。 

当你需要复制数组时，也可是直接使用Arrays.copyOf()方法，或者System.copyArray()方法，效率比遍历快多了。ArrayList就是用这个方式来复制的。 

创建ArrayList是若没有指定数组长度则默认数组长度为10，每次新增元素的相关操作都会检查是否还有数组长度是否已经用完了，若用完了，则创建一个新的数组，长度比旧的数组增长二分之一。即一次扩容后长度为15，第二次为22，以此类推。。。 

Java程序员每天都在用的ArrayList类除了自动扩容这个特性，没什么好说的。 

#### LinkedList，

List接口的链表实现，同时实现了Deque接口，也就是说LinkedList既可以当成普通的链表使用，也可以当成队列和栈来使用。

就效率而言，链表的随机插入和删除效率是高于数组的，而随机访问和修改效率则低于数组。这也是LinkedList与ArrayList的差别。

至于栈和队列的使用方法，与上面ArrayDeque演示的一致，就不重复说代码了。 


**Set集合与Map集合关系很密切，因为Set集合就是用Map集合实现的。**

在JDK的实现中，Set集合里是Map集合中Value为一个常量对象的集合。这个常量的代码如下所示。
 
```java
private static final Object PRESENT = new Object();  
```

Set集合里面持有了一个Map集合的引用，比如HashMap里面有一个HashMap的引用，然后再HashSet的构造方法里面实例化了一个对应的HashMap。 

```java
private transient HashMap<E,Object> map;  
```

Set集合的所有操作都会委托给对应的Map去实现。 所以Set集合与Map集合可以放在一起讨论。 

#### HashSet与HashMap

HashSet与HashMap，对应数据结构里面的哈希表，或者叫散列表。

根据HashMap的key对应的hashcode进行寻址，然后就可以根据key值映射的value的值了。

若是不考虑元素的存取顺序，任何时候都应该优先考虑HashSet和HashMap。毕竟哈希表在所有的数据结构中是效率最高的。 

#### LinkedHashSet与LinkedHashMap

LinkedHashSet与LinkedHashMap，在使用哈希表作为数据结构的同时维护了一个链表，这个链表记录插入数据的顺序，所以我们再遍历的时候就可以得到与插入数据顺序一致的数据了。

而其余操作是使用哈希表的数据结构，这样保证了在读取元素、插入与删除数据时的高效。 

#### TreeSet与TreeMap

TreeSet与TreeMap，使用红黑树的数据结构实现，在插入和删除数据时保证了树结构的平衡，这样在插入删除数据等操作时的效率以稳定的。

红黑树是一种特殊的二叉搜索树，这是一种排序树，所以在需要对Set和Map元素进行排序时，优先考虑TreeSet和TreeMap。默认情况下会以自然顺序对Key从小到大进行排序。 

这里需要注意的是，有两种方法可以设置TreeSet和TreeMap的比较规则，一种是构造TreeSet和TreeMap是传入自己的Comparator实现类，另一种是对插入TreeSet或TreeMap的key元素实现了Comparable接口。 

幸好，常用的基本类型和String类已经实现类Comparable接口，它们的比较规则是自然排序规则。 

若是自己定义的类没有实现Comparable接口，同时在TreeSet和TreeMap时也没有传入Comparator实现类的对象，编译时不会报错，但是在运行时会跑出ClassCastException，这是需要注意的地方。 

下面以TreeMap为例，使用自定义Key类型，分别介绍使用Comparator接口和使用Comparable接口作为比较器的用法。 

1）使用Comparator接口 

代码如下：
 
```java
public class TreeMapTest {  
    public static void main(String[] args) {  
        Map<KeyTest, Object> tm = new TreeMap<KeyTest, Object>(new KeyComparator<KeyTest>());  
        tm.put(new KeyTest(4), "4");  
        tm.put(new KeyTest(3), "3");  
        tm.put(new KeyTest(5), "5");  
        for (Entry<KeyTest, Object> entry : tm.entrySet()) {  
            System.out.println(entry.getKey().getValue() + "-->" + entry.getValue());  
        }  
    }  
}  
  
class KeyTest{  
    int value;  
  
    public KeyTest(int value){  
        this.value = value;  
    }  
      
    public int getValue() {  
        return value;  
    }  
      
}  
  
class KeyComparator<T extends KeyTest> implements Comparator<T>{  
  
    /** 
     * o1<o2返回-1，o1>o2返回1，o1=o2返回0，这是从小到大排序，反之则从大到小排序 
     */  
    @Override  
    public int compare(T o1, T o2) {  
        if(o1.getValue() < o2.getValue()){  
            return -1;  
        } else if(o1.getValue() > o2.getValue()){  
            return 1;  
        }  
        return 0;  
    }  
      
}
```

上面这段代码执行结果如下： 

![TreeMapComparetor](/images/Core_Java/JavaCollection1/TreeMapComparetor.png) 


2）使用Comparable接口 

代码如下： 

```java
public class TreeMapTest {  
    public static void main(String[] args) {  
        Map<KeyTest, Object> tm = new TreeMap<KeyTest, Object>();  
        tm.put(new KeyTest(4), "4");  
        tm.put(new KeyTest(3), "3");  
        tm.put(new KeyTest(5), "5");  
        for (Entry<KeyTest, Object> entry : tm.entrySet()) {  
            System.out.println(entry.getKey().getValue() + "-->" + entry.getValue());  
        }  
    }  
}  
  
class KeyTest implements Comparable<KeyTest>{  
    int value;  
  
    public KeyTest(int value){  
        this.value = value;  
    }  
      
    public int getValue() {  
        return value;  
    }  
  
    /** 
     * this.value<o返回-1，this.value>o返回1，this.value=o返回0，这是从小到大排序，反之则从大到小排序 
     */  
    @Override  
    public int compareTo(KeyTest o) {  
        if(this.value < o.getValue()){  
            return -1;  
        } else if(this.value > o.getValue()){  
            return 1;  
        }  
        return 0;  
    }  
      
}  
```

上面这段代码执行结果如下： 

![TreeMapComparable](/images/Core_Java/JavaCollection1/TreeMapComparable.png) 
 

建议总是优先考虑使用Comparator接口的实现类作为比较器，这样不会将比较器的实现代码侵入到JavaBean中。 

#### EnumSet与EnumMap

EnumSet和EnumMap，枚举元素在集合中的体现 

前面说到常用的Set集合使用Map集合实现的，但是就一个例外，就是EnumSet和EnumMap集合。EnumSet里面并没有持有EnumMap的引用。
 
示例代码中用到的枚举类如下： 

```java
enum EnumType{  
    TEST("test"),JDBC("jdbc"),SPRING("spring"),HIBERNATE("hibernate");  
      
    private final String value;  
      
    private EnumType(String value){  
        this.value = value;  
    }  

    public String getValue() {  
        return value;  
    }  
      
}  
```

EnumSet是抽象类，提供了一个工厂方法EnumSet.noneOf()来实例化对象。在 

EnumSet.noneOf()方法实现中，当提供的参数的枚举类型的元素少于64个时，使用RegularEnumSet实现类实例化EnumSet，否则使用JumboEnumSet实现类实例化EnumSet对象。

特别注意的是，EnumSet并没有直接提供插入元素的add()方法，而是使用多个重载的of()方法直接构造基于泛型参数类型的枚举元素，一旦of()方法调用完，就不能往EnumSet里面插入元素了。

若想使用add()方法插入枚举元素到EnumSet中，则只能使用EnumSet的子类了。 

另外EnumSet还提供了一个拷贝构造方法和拷贝方法copyOf()，用于将现有的EnumSet对象拷贝到新对象中。 

EnumSet的用法如以下代码所示： 

```java
    System.out.println("EnumSet的of()方法：");  
    EnumSet<EnumType> enumSet = EnumSet.of(EnumType.JDBC, EnumType.SPRING);  
    for (EnumType enumType : enumSet) {  
        System.out.println(enumType + "-->" + enumType.getValue());  
    }  
    System.out.println();  
    System.out.println("EnumSet的allOf()方法：");  
    enumSet = EnumSet.allOf(EnumType.class);  
    for (EnumType enumType : enumSet) {  
        System.out.println(enumType + "-->" + enumType.getValue());  
    }
```

上一段代码的执行结果如下： 

![EnumSetTest](/images/Core_Java/JavaCollection1/EnumSetTest.png) 
 

EnumMap是具体实现类，直接用构造方法实例化对象。EnumMap是只能以实例化对象时传入的枚举参数的元素作为Key的Map，若插入元素时传入的Key参数不是枚举参数的元素，则会抛出ClassCastException异常。 

EnumMap的用法如下: 

```java
    EnumMap<EnumType,Object> enumMap = new EnumMap<EnumType, Object>(EnumType.class);  
    enumMap.put(EnumType.JDBC, "jdbc元素");  
    enumMap.put(EnumType.TEST, "test元素");  
    for (Map.Entry<EnumType, Object> map : enumMap.entrySet()) {  
        System.out.println(map.getKey()+"(\""+map.getKey().getValue()+"\")"+ "-->" + map.getValue());  
    }
```

上一段代码的执行结果如下： 

![EnumMapTest](/images/Core_Java/JavaCollection1/EnumMapTest.png) 
 
