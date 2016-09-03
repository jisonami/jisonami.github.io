---
layout: post
title:  "Java集合框架（二）--Java8新增的函数式集合操作方式"
date:   2016-08-28 15:14:54
categories: Core_Java
tags: Java Collection 函数式编程
---

* content
{:toc}

这是Java集合框架第二篇，介绍关于Java8新增的函数式集合操作方式 

在我看来，Java8新增的所有特性都是为FP（函数式编程）服务的，这就要求我们要有FP思维。

长久以来，我们一直在OOP（面向对象编程）的思想下编程，OOP确实很不错，提供了清晰的接口声明，但是OOP的实现代码比较啰嗦，冗余的代码也比较多。

而FP提供了更加简洁明了的语法，但是纯用FP的代码又比较晦涩难懂。这时就有人提倡接口声明和框架分层之间使用OOP，而在具体的实现或者算法封装中使用FP，这样就把OOP和FP的优点都结合起来了。





更有甚者，有篇文章提到，接口声明使用Java提供，具体实现使用Groovy（JVM上一门动态语言，也包含了函数式的特性，还包含了元编程）实现。 

参考这篇文章： 
[Java与groovy混编 —— 一种兼顾接口清晰和实现敏捷的开发方式](https://segmentfault.com/a/1190000002601659 ) 

这涉及到JVM多语言混合编程，提高了编程的复杂性，其实我们使用Java8的新特性也可以做到函数式编程了。

而在Java8中，新增了很多函数式的特性，比如lambda表达式、高阶函数、闭包等，尽管依旧没有提供函数柯里化与偏函数（函数部分调用）的特性。 

### 简单说一说Java8的新特性 


解释一下Java8中新增的函数式特性的概念 

#### 高阶函数

高阶函数：一种可以将函数作为参数传入或者将函数作为返回值的函数。 

#### lambda表达式

lambda表达式：一种将类似数学表达式赋值给变量的语法，这个数学表达式其实就是被语法糖简化了的函数。

然后将这个lambda表达式作为参数或返回值的函数就是高阶函数。 

lambda有点像Java8之前的匿名内部类，其实不是。它们在JVM中的底层实现方式是不一样的，所以造成的结果也不完全一样。

lambda与匿名内部类一个明显的区别就是this指针的引用。匿名内部类的this是指向该匿名内部类本身，而lambda则是指向使用该lambda表达式的对象。

Java8的lambda表达式是作为实现只有单个抽象方法的接口来使用的。这种接口也叫函数式接口。 

#### 闭包

闭包：一种变量作用域，其实就是在lambda表达式中引用了外部变量时，这个外部变量自动变成final修饰的不可变变量了。

也就是说这个变量在被赋予初始值后，不管是在lambda表达式外部还是内部都不可改变了。 

#### 重新设计的接口语法

Java8的接口语法被重新设计了，原本接口是全部都是抽象函数的OOP契约，现在接口中可以有很多default默认方法（其实就是实例方法）和static静态方法。

其实变得跟抽象类的作用差不多了。不过永远记住default方法只在重构代码时才考虑引入，否则会容易造成多重继承代码无法编译通过的情况。 

#### 流式编程

Java8的流式编程，这个才是Java8函数编程的核心，通过在集合框架的顶层接口中新增default方法的方式，为原本无法扩展的集合API提供了函数式编程的入口方法，比如stream（）方法和parallelStream()方法。

此外在其它的API中也提供了流式编程的入口，这里只介绍Java集合方面的。 

### Java8新的集合操作方式 

在做了这么多啰嗦的介绍之后，终于进入本部分的正题了。 

#### Iterator接口中新增的foreach()默认方法 

先看下foreach方法的源代码： 

```java
default void forEach(Consumer<? super T> action) {  
        Objects.requireNonNull(action);  
        for (T t : this) {  
            action.accept(t);  
        }  
}  
```

这个默认方法接受一个函数式接口Consumer作为参数，所以我们可以使用lambda作为参数，这个方法其实就是将集合遍历了一下，然后对每个集合元素进行了参数中lambda表达式的执行。 

其实就是将遍历的行为交给了forEach()方法，我们只需要提供遍历的行为即可。 

在没有forEach()方法之前，我们是这样进行遍历的 

```java
    for (Integer e : ad) {  
        System.out.println(e);  
    }  
```

而使用forEach()方法则把三行代码变成了一行 

```java
ad.forEach(e -> System.out.println(e));  
```

实际上我们是实现了源码中的action.accept(t);这一行代码。 

好吧，我承认这个forEach()方法好像并没有什么用。 

#### Collection接口新增的removeIf()默认方法 

先看一下removeIf()方法的源代码 

```java
default boolean removeIf(Predicate<? super E> filter) {  
        Objects.requireNonNull(filter);  
        boolean removed = false;  
        final Iterator<E> each = iterator();  
        while (each.hasNext()) {  
            if (filter.test(each.next())) {  
                each.remove();  
                removed = true;  
            }  
        }  
        return removed;  
}  
```

其实removeIf()方法就是帮我们把集合遍历了一把，然后把满足条件的元素删掉了。 

没有removeIf()方法之前，我们是这样做的 

首先有这么一个集合 

```java
    List<Integer> al = new ArrayList<Integer>();  
    al.add(4);  
    al.add(3);  
    al.add(5);  
```

然后我们要遍历和删除大于3的元素 

```java
    Iterator<Integer> it = al.iterator();  
    while(it.hasNext()){  
        if(it.next()>3){  
            it.remove();  
        }  
    }  
```

而使用removeIf()方法代码则一行代码就解决了 

```java
al.removeIf(e -> e>3);  
```

#### Collection接口新增的stream()和parallelStream()方法 

stream()和parallelStream()方法都是获取了一个Stream对象，之后的编程方式是一样。

不同的是stream()得到的是串行流，也就是流对象的操作是单线程执行的。

parallelStream()得到的是并行流，也就是流对象的操作可能是多线程同时进行的，然后将结果合并到一起。 

永远记住Java8的流式编程没有改变原来的集合类，而是在流式方法调用之后产生了新的集合或者结果。 

这里以stream()为例介绍函数式编程的三板斧（filter、map、reduce） 

其实就是筛选、映射、合并。 

还是先以面向对象的例子说起 

如果我们要在一个集合中遍历并筛选出部分元素进行做乘2操作后求和，之前我们是这样做的 

先有一个集合 

```java
    List<Integer> al = new ArrayList<Integer>();  
    al.add(4);  
    al.add(3);  
    al.add(5);  
    al.add(7);  
    al.add(6);  
```

筛选大于4的元素记性乘2操作后求和 

```java
    int sum = 0;  
    for (Integer e : al) {  
        if(e >4){  
            sum = sum + e*2;  
        }  
    }  
```

如果使用stream()的方式来操作是这样的 

```java
int sum = al.stream()  
    .filter(e -> e>4)  
    .map(e -> e*2)  
    .reduce((e1,e2) -> (e1+e2)).get();  
```

有些人认为使用collect来替代reduce好一些，其实是一样的，只是少了个get()操作而已。 

```java
int sum = al.stream()  
    .filter(e -> e>4)  
    .map(e -> e*2)  
    .collect(Collectors.summingInt(e -> e));  
```

#### List接口新增的sort()默认方法 

先看下源码 

```java
default void sort(Comparator<? super E> c) {  
    Object[] a = this.toArray();  
    Arrays.sort(a, (Comparator) c);  
    ListIterator<E> i = this.listIterator();  
    for (Object e : a) {  
        i.next();  
        i.set((E) e);  
    }  
}  
```

可以看到sort方法就是接受了一个Comparator比较器对象，然后将List集合转换成数组，然后再调用数组工具类Arrays的sort()方法进行排序，最后把排好序的数组遍历并赋值到原来的List集合上。 

原来我们是使用Collections.sort()方法来进行排序的，现在这个Collections.sort()方法直接将实现委托到List接口的默认方法sort()下了。其实是一样的。 

在Java8里面唯一比以前方便的就是可以直接使用lambda表达式传入Comparator比较器的实现参数了。 

假设我们在List存放的元素是下面这个类 

```java
class KeyTest{  
    int value;  
  
    public KeyTest(int value){  
        this.value = value;  
    }  
      
    public int getValue() {  
        return value;  
    }  
      
}  
```

然后往List插入几个KeyTest元素 

```java
    List<KeyTest> al = new ArrayList< KeyTest>();  
    al.add(new KeyTest(4));  
    al.add(new KeyTest(3));  
    al.add(new KeyTest(5));  
```

然后排序 

```java
al.sort((e1,e2) -> Integer.valueOf(e1.getValue()).compareTo(e2.getValue()));  
```

当然，sort()方法也可以传入null值作为参数，这要求插入List中的元素必须实现Comparable接口，即 

如果KeyTest实现了Comparable接口 

```java
class KeyTest implements Comparable<KeyTest>{  
        int value;  
  
        public KeyTest(int value){  
            this.value = value;  
        }  
          
        public int getValue() {  
            return value;  
        }  
  
        @Override  
        public int compareTo(KeyTest o) {  
            return Integer.valueOf(this.value).compareTo(o.value);  
        }  
    }  
```

那么排序时就可以这样 

```java
al.sort(null);  
```

至此，Iterator继承系列的接口新增的默认方法就介绍完了，接下来说一说Map接口新增的一些默认方法，不过Map接口没有提供转换成Stream接口的方法。 

#### Map接口新增的forEach()默认方法 

这个Map接口的forEach()默认方法与Collection接口的forEach()方法用法是一致的，不过这里遍历的是Map的entrySet，看下源码就知道了。 

```java
default void forEach(BiConsumer<? super K, ? super V> action) {  
        Objects.requireNonNull(action);  
        for (Map.Entry<K, V> entry : entrySet()) {  
            K k;  
            V v;  
            try {  
                k = entry.getKey();  
                v = entry.getValue();  
            } catch(IllegalStateException ise) {  
                // this usually means the entry is no longer in the map.  
                throw new ConcurrentModificationException(ise);  
            }  
            action.accept(k, v);  
        }  
}  
```

我们要实现的方法是action.accept(k, v); 

```java
map.forEach((k, v) -> System.out.println(k + "->" + v));  
```

#### Map接口新增的replaceAll()默认方法 

这个方法的逻辑其实是把entrySet遍历了一把，然后把所有的value替换成lambda表达式计算的结果，key值保存不变。 

先看下源码 

```java
default void replaceAll(BiFunction<? super K, ? super V, ? extends V> function) {  
        Objects.requireNonNull(function);  
        for (Map.Entry<K, V> entry : entrySet()) {  
            K k;  
            V v;  
            try {  
                k = entry.getKey();  
                v = entry.getValue();  
            } catch(IllegalStateException ise) {  
                // this usually means the entry is no longer in the map.  
                throw new ConcurrentModificationException(ise);  
            }  
  
            // ise thrown from function is not a cme.  
            v = function.apply(k, v);  
  
            try {  
                entry.setValue(v);  
            } catch(IllegalStateException ise) {  
                // this usually means the entry is no longer in the map.  
                throw new ConcurrentModificationException(ise);  
            }  
        }  
}  
```

关键的代码是这行 

```java
v = function.apply(k, v);  
```

我们传入的lambda参数需要实现这个apply()方法 

有这么一个Map集合 

```java
    Map<String, String> map = new HashMap<String, String>();  
    map.put("test", "value test");  
    map.put("jdbc", "value jdbc");  
    map.put("spring", "value spring");  
```

我们需要对这个map的所有value的”value”字符串变成”element”，那么 

```java
map.replaceAll((k, v) -> v.replaceAll("value", "element"));  
```

使用之前的forEach()方法遍历就可以看到结果 

![HashMapForEach](/images/Core_Java/JavaCollection2/HashMapForEach.png)


#### Map接口其余的默认方法 

以下几个方法均是对value取值和设置的方法 

* 若key值对应的value为null，则返回defaultValue 
```java
V getOrDefault(Object key, V defaultValue)
```

* 若key值对应为value为null，设值为value，否则不操作 
```java
V putIfAbsent(K key, V value)  
```

* 若key值对应的value为null，根据mappingFunction计算的值设值，否则不操作 
```java
V computeIfAbsent(K key, Function<? super K, ? extends V> mappingFunction)
```

* 若key值对应的value不为空，根据remappingFunction计算的值设值，否则不操作 
```java
V computeIfPresent(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction)
```

* 根据remappingFunction计算的值对key对应的value设值，若计算的值为null，则删除key-value键值对 
```java
V compute(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction)  
```

* 若key对应的value为null，则直接将第二个参数value对key对应的value设值。若key对应的value不为null，根据remappingFunction计算的值对key对应的value设值。若计算的值为null，则删除key-value键值对。 
```java
V merge(K key, V value, BiFunction<? super V, ? super V, ? extends V> remappingFunction)  
```

最后，Java8对集合框架新增的默认方法大部分都是传入lambda表达式作为参数的高阶函数。

刚接触函数式编程的同学可能觉得不容易看懂，其实对函数式编程的概念理解清楚之后，直接看这些API的源代码是比较容易的。 