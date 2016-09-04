---
layout: post
title:  "Java的反射"
date:   2016-03-19 22:27:54
categories: Core_Java
tags: Java 反射
---

* content
{:toc}

我们知道，Java的类型信息分为编译时类型信息和运行时类型信息，而反射就是Java提供的对运行时类型信息获取和操作的机制。 

那么Java的类型信息有什么呢？一个Java的类主要包括两个元素，即是成员变量和成员方法。

成员变量包括实例成员变量和静态成员变量，而成员方法也有实例成员方法和静态成员方法，构造方法则是特殊的实例成员方法。

而反射的主要作用是能够在运行时获取一个Class的各个元素的结构，但无法更改这些元素的结构。




这些元素就是前面说的成员变量和成员方法，并且对于成员变量，反射可以对其进行设值和取值，对于成员方法，反射可以直接对其进行方法调用，而对于特殊的成员方法构造方法而言，反射可以调用构造方法元素的newInstance（）方法直接实例化一个该类的对象，而不需要使用new的方式。 

在Java的反射API里，Java的类型信息体现为几个元素，这些元素在java.lang.reflect包下体现为以下几个类： 

Field	描述字段的结构信息 

Constructo描述构造方法的结构信息 

Method	描述普通方法的结构信息 

在Java8以后，新增了一个Parameter类，用于描述构造方法和普通方法的参数结构信息。 

目前还有很多企业还是使用Java6的API，特别是做企业信息化系统的企业，至少前公司用的就是Java6的API。但是Java8的API添加了挺多API。所以下面分别提供了Java6和Java8里关于反射API的类图，也好有个对比，当然重点还是介绍Java8的API。 

Java6的反射类图 

![Java6的反射类图](/images/Core_Java/JavaReflection/Java6ReflectionClassDiagram.jpg)

Java8的反射类图 

![Java8的反射类图](/images/Core_Java/JavaReflection/Java8ReflectionClassDiagram.jpg)

看到上面两个类图，Type接口及其子接口是与泛型相关的，AnnotatedType接口及其子接口则是与注解相关的，我们主要关注Field、Executable、Constructor、Method和Parameter这几个类，还有Class这个反射的入口类。 

假设有这样的接口和类 

```java
interface MyInterface{  
    void method(String arg0, Integer arg1);  
}  
public class JavaReflect implements MyInterface{  
    public JavaReflect(){}  
    public JavaReflect(String name){  
        System.out.println("实例化了一个对象，传入参数name为：" + name);  
    }  
    public String field;  
    @Override  
    public void method(String arg0, Integer arg1){  
        System.out.println("调用了method方法，参数值为：" + arg0 + "和" + arg1);  
    }  
}  
```

关于Class这个反射的入口类，我们有三种方式可以获得。 

```java
    Class<?> clazz = Class.foName(“JavaReflect”);  
```

```java
    Class<?> clazz = JavaReflect.class;
```

```java
    JavaReflect reflect = new JavaReflect ();  
    Class<?> clazz = reflect.getClass();  
```

通过这三种方式得到Class对象之后，我们就可以访问这个类型的运行时信息了。 

利用这些信息，我们可以做到以下几点。 

一．	实例化对象 
反射实例化对象的方式有两种，一种是调用Class类的newInstance()方法，该方法会调用无参构造方法实例化一个对象。 

```java
Object obj = clazz.newInstance();  
```

另一种则是先调用Class类的getConstructor(Class<?>... parameterTypes)方法获取对应的Constructor对象，然后调用Constructor.newInstance(Object … args)方法实例化一个对象。 

```java
Constructor<?> constructor = clazz.getConstructor(String.class);  
Object obj = constructor.newInstance("一个参数");  
```

二．	对字段取值和设值 

反射对字段取值和设值都需要先获取Field类的实例，然后调用Field类的取值和设值方法进行取值和设值。 

Class类有两种获取Field类实例的方法，一个是getField(String name)方法，另一个是Field[] getFields()方法先获取Field[]数组，然后遍历该数组，根据字段名来判断所需的字段。 

Field类的设值方法是set(Object obj, Object value)，而原始类型则是setXXX(Object obj, Object value) 

Field类的取值方法是get(Object obj)，而原始类型则是getXXX(Object obj) 
代码示例 

```java
Field field = clazz.getField("field");  
    field.set(obj, "给字段设值");  
    Object fieldValue = field.get(obj);  
```

### 调用方法 

反射调用Method getMethod(String name, Class<?>... parameterTypes)方法需要先获取Method类的实例，然后调用Method类的Object invoke(Object obj, Object... args)方法调用对应的方法。

**因为方法存在重载，所以不能通过获取所有Method[]数组然后循环比较方法名来取得特定的方法** 

```java
Method method = clazz.getMethod("method", String.class, Integer.class);  
method.invoke(obj, "字符串", 1);  
```

### 反射与数组 

在Java反射中，数组使用Array描述，Array类主要有三类方法，全部都是静态方法。 

1.	实例化数组的方法 
newInstance(Class<?> componentType, int... dimensions) 创建一个具有指定的组件类型和维度的新数组。 

```java
Object array2 = Array.newInstance(clazz, 2, 3);  
```

newInstance(Class<?> componentType, int length) 创建一个具有指定的组件类型和长度的新数组。 
```java
Object array1 = Array.newInstance(clazz, 2);  
```

2.	对数组元素设值的方法 
set(Object array, int index, Object value) 将指定数组对象中索引组件的值设置为指定的新值，若是原始类型数组则是setXXX(Object array, int index, Object value)方法。 

```java
Array.set(array1, 1, obj);  
Object obj2 = clazz.newInstance();  
    obj2.getClass().getField("field").set(obj2, "反射处理二维数组");  
    Array.set(Array.get(array2, 1), 2, obj2);  
```

3.	对数组元素取值的方法 
get(Object array, int index) 返回指定数组对象中索引组件的值，若是原始类型数组则是getXXX(Object array, int index)方法。 

```java
Object obj1 = Array.get(array1, 1);  
Object obj3 = Array.get(Array.get(array2, 1), 2);  
```

### 反射与动态代理 

Java的动态代理使用起来很简单，使用Proxy类和InvocationHandler接口就可以实现。Java实现的动态代理对象就是实现了指定n个接口的类的一个对象，使用代理对象调用任何方法时，都会替换为InvocationHandler接口实现类重写的invoke方法。 

1.	实现InvocationHandler接口，重写invoke(Object proxy, Method method, Object[] args)方法，该方法就是代理方法，将会替换掉代理对象被调用的方法。 

```java
public class InvocationHandlerImpl implements InvocationHandler{  
    private Object target;  
    public InvocationHandlerImpl(Object target){  
        this.target = target;  
    }  
    @Override  
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {  
        System.out.println("方法调用前，相当于Spring AOP的前置通知");  
        method.invoke(target, args);  
        System.out.println("方法调用后，相当于Spring AOP的后置通知");  
        return null;  
    }  
}  
```

2.	调用Proxy的静态方法创建代理的对象，有以下两种方法 

一种方法是使用Proxy的Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)方法直接创建代理对象， 

```java
Object proxyObject = Proxy.newProxyInstance(clazz.getClassLoader(),   
                new Class<?>[]{MyInterface.class},   
                new InvocationHandlerImpl(obj));  
```

另一种是先调用Class<?> getProxyClass(ClassLoader loader, Class<?>... interfaces)获取代理类，然后获取代理类的含有一个InvocationHandler类型的参数的构造方法，再使用这个构造方法实例化一个代理对象。 

```java
Class<?> proxyClass = Proxy.getProxyClass(clazz.getClassLoader(),  MyInterface.class);  
        Object proxyObject = proxyClass.getConstructor(InvocationHandler.class).newInstance(new InvocationHandlerImpl(obj));  
```

3.	使用代理的对象调用目标方法。 

```java
((MyInterface) proxyObject).method("使用动态代理调用该方法", 0);  
```

六．	反射与泛型 
参考我的博文Java反射获取实际泛型类型参数 
七．	反射与注解 
参考我的博文Java注解知识点总结 

上面的代码完整的示例如下： 

```java
import java.lang.reflect.Array;  
import java.lang.reflect.Constructor;  
import java.lang.reflect.Field;  
import java.lang.reflect.Method;  
import java.lang.reflect.Proxy;  
  
interface MyInterface{  
    void method(String arg0, Integer arg1);  
}  
public class JavaReflect implements MyInterface{  
    public JavaReflect(){}  
    public JavaReflect(String name){  
        System.out.println("实例化了一个对象，传入参数name为：" + name);  
    }  
    public String field;  
    @Override  
    public void method(String arg0, Integer arg1){  
        System.out.println("调用了method方法，参数值为：" + arg0 + "和" + arg1);  
    }  
      
    public static void main(String[] args) throws Exception {  
        // 获取Class对象，这个是Java反射的入口  
        Class<?> clazz = JavaReflect.class;  
        // 实例化对象  
        // Object obj = clazz.newInstance();  
        Constructor<?> constructor = clazz.getConstructor(String.class);  
        Object obj = constructor.newInstance("一个参数");  
        // 字段设值和取值  
        Field field = clazz.getField("field");  
        field.set(obj, "给字段设值");  
        Object fieldValue = field.get(obj);  
        System.out.println(fieldValue);  
        // 方法调用  
        Method method = clazz.getMethod("method", String.class, Integer.class);  
        method.invoke(obj, "字符串", 1);  
        // 实例化数组  
        Object array1 = Array.newInstance(clazz, 2);  
        Object array2 = Array.newInstance(clazz, 2, 3);  
        // 数组设值和取值  
        Array.set(array1, 1, obj);  
        Object obj1 = Array.get(array1, 1);  
        System.out.println("array1的第一个元素的field字段的值为：" + obj1.getClass().getField("field").get(obj1));  
        // 对array2二维数组和多位数组的设值要先取到对应的一维数组  
        Object obj2 = clazz.newInstance();  
        obj2.getClass().getField("field").set(obj2, "反射处理二维数组");  
        Array.set(Array.get(array2, 1), 2, obj2);  
        Object obj3 = Array.get(Array.get(array2, 1), 2);  
        System.out.println("array1的第一个元素的field字段的值为：" + obj3.getClass().getField("field").get(obj3));  
        // Java反射与动态代理  
//      Class<?> proxyClass = Proxy.getProxyClass(clazz.getClassLoader(), MyInterface.class);  
//      Object proxyObject = proxyClass.getConstructor(InvocationHandler.class).  
//              newInstance(new InvocationHandlerImpl(obj));  
        Object proxyObject = Proxy.newProxyInstance(clazz.getClassLoader(),   
                new Class<?>[]{MyInterface.class},   
                new InvocationHandlerImpl(obj));  
        ((MyInterface) proxyObject).method("使用动态代理调用该方法", 0);  
          
    }  
}  
  
public class InvocationHandlerImpl implements InvocationHandler{  
    private Object target;  
    public InvocationHandlerImpl(Object target){  
        this.target = target;  
    }  
    @Override  
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {  
        System.out.println("方法调用前，相当于Spring AOP的前置通知");  
        method.invoke(target, args);  
        System.out.println("方法调用后，相当于Spring AOP的后置通知");  
        return null;  
    }  
}  
```

代码执行结果为： 

![JavaReflectionTest](/images/Core_Java/JavaReflection/JavaReflectionTest.jpg)