---
layout: post
title:  "Java反射获取实际泛型类型参数"
date:   2016-03-14 22:27:54
categories: Core_Java
tags: Java 反射 泛型
---

* content
{:toc}

我们知道，一个Java的类主要包括两个元素，即是成员变量和成员方法。成员变量包括实例成员变量和静态成员变量，而成员方法也有实例成员方法和静态成员方法，构造方法则是特殊的成员方法。

而反射的主要作用是能够在运行时获取一个Class的各个元素的结构，但无法更改这些元素的结构。

这些元素就是前面说的成员变量和成员方法，并且对于成员变量，反射可以对其进行设值和取值，对于成员方法，反射可以直接对其进行方法调用，而对于特殊的成员方法构造方法而言，反射可以调用构造方法元素的newInstance（）方法直接实例化一个该类的对象，而不需要使用new的方式。 

那么现在问题来了，在使用反射获取一个Class的各个元素的结构时，哪些地方可以直接获取得到泛型类型参数，并且能够将该泛型类型参数直接实例化成一个对象呢？ 




### 获取泛型的三种形式

在对反射进行深入研究后发现，有以下三种情况可以实现： 

1. 成员变量类型的泛型参数。 

2. 成员方法返回值的泛型参数。 

3. 成员方法参数类型的泛型参数，包括构造方法。 

在说到泛型类型参数时，要先说一下Type接口，Type接口只有一个实现类Class，但是有四个子接口，这四个Type子接口描述了**Java泛型的四种形式**。分别是： 

1. GenericArrayType	该接口表示一种数组类型，其组件类型为参数化类型或类型变量，如参数化类型数组Map<String, Date>[] mapArr，还有类型变量数组T[] tArr 

2. ParameterizedType	该接口表示参数化类型，如 Collection<String>。 

3. TypeVariable<D>	该接口是各种类型变量的公共高级接口，可以表示泛型声明的参数类型（不存在的类型），如 class ClassName<T>和Collection<T>，这里的T就是一个类型变量。 

4. WildcardType	表示一个通配符类型表达式，如 ?、? extends Number 或 ? super Integer。 

**在Java泛型的这四种形式中，类型变量和类型变量数组还有通配符类型表达式都是取不到泛型参数的实际类型的，只有泛型的参数化类型和参数化类型数组中的实际类型可以得到实际类型，并可用于实例化，也就是说只有前两种情况GenericArrayType和ParameterizedType才有可能得到泛型的实际类型。** 

下面结合例子详细说明可以取到泛型实际类型的三种情况： 

在这三种情况中，第一步都是得到一个Type类型或Type数组。 

### 成员变量类型的泛型参数。 

Field类下有个Type getGenericType()方法可以获取泛型类，返回类型为Type，代码如下： 

```java
Type type = field. getGenericType();  
```

### 成员方法返回值的泛型参数。 

Method类下有个Type getGenericReturnType()方法可以获取成员方法返回值的泛型类，返回类型为Type，代码如下： 

```java
Type type = method. getGenericReturnType();  
```

### 成员方法参数类型的泛型参数，包括构造方法。 

Method类下有个Type[] getGenericParameterTypes()方法可以获得成员方法各个参数的泛型类的数组，返回值为Type[]，（Constructor类里面也有一个getGenericParameterTypes()方法，也可以使用相同的方式取到构造器参数里的实际泛型参数）代码如下： 

```java
Type[] typeArr = method. getGenericParameterTypes();  
```

另一种方式是使用Method类下的Parameter[] getParameters()方法先获取各个参数组成的数组Parameter []，然后再使用Parameter类下的Type getParameterizedType()方法获取参数的泛型类，返回值为Type。（这种方式是Java8以后才有的，因为Parameter是Java8才新增的表示方法参数元素的类型。getParameters()方法其实是Method继承自Executable抽象类里面的方法，Executable抽象类是也是Java8新增的，是Method和Constructor两个类的基类，所以构造方法也可以使用这个方式得到构造器参数里的实际泛型参数） 

代码如下： 

```java
Parameter [] parameterArr = Method. getParameters();  
Type[] typeArr = new Type[parameterArr.length];  
for(i=0;i<parameterArr.length;i++){  
    typeArr[i] = parameterArr[i]. getParameterizedType();  
}  
```

在第一步得到的Type类型的对象后，这个对象有可能是Type接口的四个子接口实现类的实例。在使用测试代码测试后发现，这个对象只会是GenericArrayType和ParameterizedType两个接口的实现类的实例。 

所以第二步我们要做的就是将其封装成一个递归方法，如以下代码所示： 

```java
void instanceActualTypeArguments(Type type) throws Exception{  
        System.out.println("该类型是"+ type);  
        // 参数化类型  
        if ( type instanceof ParameterizedType ) {  
            Type[] typeArguments = ((ParameterizedType)type).getActualTypeArguments();  
            for (int i = 0; i < typeArguments.length; i++) {  
                // 类型变量  
                if(typeArguments[i] instanceof TypeVariable){  
                    System.out.println("第" + (i+1) +  "个泛型参数类型是类型变量" + typeArguments[i] + "，无法实例化。");  
                }   
                // 通配符表达式  
                else if(typeArguments[i] instanceof WildcardType){  
                    System.out.println("第" + (i+1) +  "个泛型参数类型是通配符表达式" + typeArguments[i] + "，无法实例化。");  
                }  
                // 泛型的实际类型，即实际存在的类型  
                else if(typeArguments[i] instanceof Class){  
                    System.out.println("第" + (i+1) +  "个泛型参数类型是:" + typeArguments[i] + "，可以直接实例化对象");  
                }  
            }  
        // 参数化类型数组或类型变量数组  
        } else if ( type instanceof GenericArrayType) {  
            System.out.println("该泛型类型是参数化类型数组或类型变量数组，可以获取其原始类型。");  
            Type componentType = ((GenericArrayType)type).getGenericComponentType();  
            // 类型变量  
            if(componentType instanceof TypeVariable){  
                System.out.println("该类型变量数组的原始类型是类型变量" + componentType + "，无法实例化。");  
            }   
            // 参数化类型，参数化类型数组或类型变量数组  
            // 参数化类型数组或类型变量数组也可以是多维的数组，getGenericComponentType()方法仅仅是去掉最右边的[]  
            else {  
                // 递归调用方法自身  
                instanceActualTypeArguments(componentType);  
            }  
        } else if( type instanceof TypeVariable){  
            System.out.println("该类型是类型变量");  
        }else if( type instanceof WildcardType){  
            System.out.println("该类型是通配符表达式");  
        } else if( type instanceof Class ){  
            System.out.println("该类型不是泛型类型");  
        } else {  
            throw new Exception();  
        }  
    }  
```

从上面的分析可知，我们使用反射时获取泛型类型参数时只需要两步就行了。在实际编写代码的时候，第二步需要根据需求进行相应的更改。 

下面用一个完整的程序测试前面说到的反射时获取泛型类型参数的方法。 

```java
import java.io.IOException;  
import java.lang.reflect.Constructor;  
import java.lang.reflect.Field;  
import java.lang.reflect.GenericArrayType;  
import java.lang.reflect.Method;  
import java.lang.reflect.ParameterizedType;  
import java.lang.reflect.Type;  
import java.lang.reflect.TypeVariable;  
import java.lang.reflect.WildcardType;  
import java.util.List;  
import java.util.Map;  
  
public class TypeArguments<T> {  
    public TypeArguments(){}  
    public <E> TypeArguments(E e){}  
    public Map<T, String> genericField;  
    public <B> Map<Integer, String>[] genericMethod(List<? extends Integer> list, List<String> list2, String str, B[] tArr) throws IOException, NoSuchMethodException{  
        return null;  
    }  
      
    public static void main(String[] args) throws Exception {  
        Class<?> clazz = TypeArguments.class;  
          
        System.out.println("一．  成员变量类型的泛型参数");  
        Field field = clazz.getField("genericField");  
        Type fieldGenericType = field.getGenericType();  
        instanceActualTypeArguments(fieldGenericType);  
        System.out.println();  
          
        System.out.println("二．  成员方法返回值的泛型参数。");  
        Method method = clazz.getMethod("genericMethod", new Class<?>[]{List.class, List.class, String.class, Object[].class});  
        Type genericReturnType = method.getGenericReturnType();  
        instanceActualTypeArguments(genericReturnType);  
        System.out.println();  
          
        System.out.println("三．  成员方法参数类型的泛型参数。");  
        Type[] genericParameterTypes = method.getGenericParameterTypes();  
        for (int i = 0; i < genericParameterTypes.length; i++) {  
            System.out.println("该方法的第" + (i+1) + "个参数：");  
            instanceActualTypeArguments(genericParameterTypes[i]);  
        }  
        System.out.println();  
          
        System.out.println("三．  构造方法参数类型的泛型参数。");  
        Constructor<?> constructor = clazz.getConstructor(new Class<?>[]{Object.class});  
        Type[] constructorParameterTypes = constructor.getGenericParameterTypes();  
        for (int i = 0; i < constructorParameterTypes.length; i++) {  
            System.out.println("该构造方法的第" + (i+1) + "个参数：");  
            instanceActualTypeArguments(constructorParameterTypes[i]);  
        }  
        System.out.println();  
    }  
      
    /** 
     * 实例化泛型的实际类型参数 
     * @param type 
     * @throws Exception  
     */  
    private static void instanceActualTypeArguments(Type type) throws Exception{  
        System.out.println("该类型是"+ type);  
        // 参数化类型  
        if ( type instanceof ParameterizedType ) {  
            Type[] typeArguments = ((ParameterizedType)type).getActualTypeArguments();  
            for (int i = 0; i < typeArguments.length; i++) {  
                // 类型变量  
                if(typeArguments[i] instanceof TypeVariable){  
                    System.out.println("第" + (i+1) +  "个泛型参数类型是类型变量" + typeArguments[i] + "，无法实例化。");  
                }   
                // 通配符表达式  
                else if(typeArguments[i] instanceof WildcardType){  
                    System.out.println("第" + (i+1) +  "个泛型参数类型是通配符表达式" + typeArguments[i] + "，无法实例化。");  
                }  
                // 泛型的实际类型，即实际存在的类型  
                else if(typeArguments[i] instanceof Class){  
                    System.out.println("第" + (i+1) +  "个泛型参数类型是:" + typeArguments[i] + "，可以直接实例化对象");  
                }  
            }  
        // 参数化类型数组或类型变量数组  
        } else if ( type instanceof GenericArrayType) {  
            System.out.println("该泛型类型是参数化类型数组或类型变量数组，可以获取其原始类型。");  
            Type componentType = ((GenericArrayType)type).getGenericComponentType();  
            // 类型变量  
            if(componentType instanceof TypeVariable){  
                System.out.println("该类型变量数组的原始类型是类型变量" + componentType + "，无法实例化。");  
            }   
            // 参数化类型，参数化类型数组或类型变量数组  
            // 参数化类型数组或类型变量数组也可以是多维的数组，getGenericComponentType()方法仅仅是去掉最右边的[]  
            else {  
                // 递归调用方法自身  
                instanceActualTypeArguments(componentType);  
            }  
        } else if( type instanceof TypeVariable){  
            System.out.println("该类型是类型变量");  
        }else if( type instanceof WildcardType){  
            System.out.println("该类型是通配符表达式");  
        } else if( type instanceof Class ){  
            System.out.println("该类型不是泛型类型");  
        } else {  
            throw new Exception();  
        }  
    }  
}  
```

代码的运行结果如下： 

![JavaReclectionAndGenericTypeTest](/images/Core_Java/JavaReflectionAndGenericType/JavaReclectionAndGenericTypeTest.png)

在我写完这篇博客的时候，我发现了这么一句话，

>Java泛型有这么一种规律：  
位于声明一侧的，源码里写了什么到运行时就能看到什么；  
位于使用一侧的，源码里写什么到运行时都没了。

这句话原文出自RednaxelaFX的博客"[答复: Java获得泛型类型](http://rednaxelafx.iteye.com/blog/586212)" 
 
而我这里探讨的泛型则符合位于声明一侧的。 