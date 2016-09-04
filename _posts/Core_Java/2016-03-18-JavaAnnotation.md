---
layout: post
title:  "Java的注解"
date:   2016-03-18 22:27:54
categories: Core_Java
tags: Java 注解
---

* content
{:toc}

Java的注解（Annotation）是Java5以后引入的，又叫元数据，也有人翻译成注释，用作给代码元素做标记，可以携带一些说明或配置信息，但是注解本身并不参与代码的运行，需要时必须对编写代码提取注解信息。注解可以修饰的一个类里面的各个组成元素，比如可以修饰类和接口的声明、构造方法、字段、方法还有方法参数等等，具体可以修饰什么元素得看该注解的声明。 

注解主要有三个知识点： 

一．	Java提供的注解 

二．	利用元注解自定义注解 

三．	提取注解信息 




注解使用的语法形式是 

```java
@AnnotationName  
```

或 

```java
@AnnotationName(valueName1 = value1, valueName2 = value2, …)  
```

当valueName只有一个并且就是“value”， 我们可以省略valueName，写成如下形式： 

```java
@AnnotationName(value)  
```

如不省略则如下 

```java
@AnnotationName(valueName = value)  
```

注解的用法与Java修饰符一样，一般写在修饰符的最前面，并且独占一行，因注解可能存在参数，所以我们惯常的写法如下（以声明一个类为例），当然不换行语法上也是正确的。 

```java
@AnnotationName  
Public class ClassName{  
    // …  
}  
```

### Java提供的注解 

下面介绍Java提供的五个常用的注解： 

1.	用于标记Java代码元素已过时的注解：@Deprecated 

用 @Deprecated 注解的程序元素，不鼓励程序员使用这样的元素，通常是因为它很危险或存在更好的选择，这种程序元素很可能下一代API就取消掉了。在使用不被赞成的程序元素或在不被赞成的代码中执行重写时，编译器会发出警告。@Deprecated注解基本可以用于任何程序元素。 

2.	用于标记重写基类方法的注解：@Override 

表示一个方法声明打算重写基类中的另一个方法声明。如果方法利用此注解类型进行注解但没有重写基类方法，则编译器会生成一条错误消息。@Override注解只能用于方法生命时。 

3.	用于标记抑制编译器警告的注解：@SuppressWarnings 

用 @ SuppressWarnings注解的程序元素可能会产生编译器警告，而使用了这个注解则表示取消了编译器的警告信息。@SuppressWarnings注解基本可以用于任何程序元素。 

4.	用于标记方法编译时可能发生堆污染警告的注解（Java7新增注解）：@SafeVarargs 

在将非泛型对象赋值给泛型对象时会产生泛型擦除，而在赋值后的非泛型对象上取值操作时会产生运行时异常，称之为堆污染（Heap Pollution）。使用@SafeVarargs注解修饰后，将不会产生堆污染警告。@SafeVarargs只能用在构造方法或普通方法声明时。事实上在使用Eclipse编写Java代码时对@SafeVarargs注解修饰的方法会报错，一般会使用@SuppressWarnings("unchecked")替代。 

5.	用于标记函数式接口的注解（Java8新增）：@FunctionalInterface 

Java8新增了函数式编程，只含有一个抽象方法的接口在Java8中称为函数式接口，使用@FunctionalInterface注解修饰。@FunctionalInterface注解只能用在接口声明时。 

使用示例如下： 

```java
@FunctionalInterface  
interface MyFunctionalInterface{  
    void method(List<String> ... lists );  
}  
@Deprecated  
public class AnnotationTest implements MyFunctionalInterface{  
    @SuppressWarnings({ "unused"})  
    private String field;  
    @Override  
    @SafeVarargs    // @SafeVarargs在Eclipse编辑器中不支持，直接代码提示方法签名报错  
    public void method(List<String> ... lists ) {  
        // ...  
    }  
    public static void main(String[] args) {  
        List<String> localField1 = new ArrayList<String>();  
        List<String> localField2 = new ArrayList<String>();  
        // 这一行会引发堆污染  
        new AnnotationTest().method(localField1, localField2);  
    }  
}  
```

### 利用元注解自定义注解 

Java提供了几个用来定义注解的注解，又叫元注解，只能用在类型声明上，用来定义注解的使用范围和生命周期等等。 

@Documented	用于标记一个注解是否会被javadoc提取成API文档 

@Inherited	用于标记一个注解是否会被使用的类的扩展类继承，该注解修饰的注解只能用在类的声明上才有效， 

@Retention	用于标记一个注解的声明周期，使用枚举类RetentionPolicy的三个枚举值决定，只能使用其中一个枚举值。 

SOURCE 编译器要丢弃的注解。 

CLASS 编译器将把注解记录在类文件中，但在运行时 VM 不需要保留注解。 

RUNTIME 编译器将把注解记录在类文件中，在运行时 VM 将保留注解，因此可以反射性地读取。 

@ Target	用于标记一个注解能够用于哪些程序元素，使用枚举类ElementType的枚举值决定，可以使用多个枚举值。 

ANNOTATION_TYPE 注解类型声明 

CONSTRUCTOR  构造方法声明 

FIELD  字段声明（包括枚举常量） 

LOCAL_VARIABLE  局部变量声明 

METHOD 方法声明 

PACKAGE 包声明 

PARAMETER 参数声明 

TYPE 类、接口（包括注解类型）或枚举声明 

Java8又新增了两个枚举值 

TYPE_PARAMETER Type parameter declaration（类型参数声明） 

TYPE_USE Use of a type（一个类的任意元素都可使用） 

Java8新增了两个元注解 

@ Repeatable	用于标记一个注解可以重复多次使用在一个程序元素上。 

@ Native	用于标记一个注解用在一个字段的常量值可能来自非Java代码，这是目前Java元注解中唯一一个RetentionPolicy. SOURCE级别的元注解，其余元注解都是RUNTIME级别的。平时似乎也不会用到。 

Java定义注解的语法使用@interface来定义，例如我们可以定义一个这样的注解。 

```java
@Retention(RetentionPolicy.RUNTIME)  
@Target({ElementType.FIELD, ElementType.METHOD})  
@interface MyAnnotation{  
    String value();  
}  
```

现在这个注解只能使用在字段和普通方法上，如 

```
// @MyAnnotation("该注解声明Target时没用使用TYPE枚举值，不能使用在类和接口等的声明上")  
public class AnnotationUserDefined {  
    @MyAnnotation("默认的value数据赋值可以省略value = ")  
    public String field;  
    @MyAnnotation(value = "不省略value = 得写法")  
    public void method(){  
        // ...  
    }  
}  
```

从上面的用法上可以看到，注解内声明字段的语法不再是普通字段声明的语法，而是和普通方法的声明一样。上面声明了一个value的字段，当注解声明的字段只有一个value时，注解使用时赋值操作可以省略“value = ”，当注解声明的字段名不叫value时或有多个字段（即使其中一个字段包含value），赋值操作不能省略字段名。 

如字段名不叫value 

```java
@Retention(RetentionPolicy.RUNTIME)  
@Target({ElementType.FIELD, ElementType.METHOD})  
@interface MyAnnotation{  
    String name();  
}  
```

使用时必须这样 

```java
@MyAnnotation(name = "不能省略name = ")  
```

如不是只有一个value字段时 

```java
@Retention(RetentionPolicy.RUNTIME)  
@Target({ElementType.FIELD, ElementType.METHOD})  
@interface MyAnnotation{  
    String value();  
    String name();  
}  
```

使用时必须这样 

```java
@MyAnnotation(value = "不能省略value= ", name = "不能省略name = ")  
```

若是在@MyAnnotation注解的声明上使用@Inherited修饰，则使用@MyAnnotation注解修饰的类的扩展类若没有使用该注解修饰，也自动继承了@MyAnnotation注解。同时，在@MyAnnotation声明上，@Target必须加上ElementType.TYPE枚举值或ElementType.TYPE_USE，否则该注解不起作用。 

```java
@Retention(RetentionPolicy.RUNTIME)  
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.TYPE})  
@interface MyAnnotation{  
    String value();  
}  
```

那么@MyAnnotation就可以使用在类型声明上了。 

```java
@MyAnnotation("在类型声明上使用该注解")  
public class AnnotationUserDefined {  
    @MyAnnotation("默认的value数据赋值可以省略value = ")  
    public String field;  
    @MyAnnotation(value = "不省略value = 得写法")  
    public void method(){  
        // ...  
    }  
}  
// 此处自动继承了基类AnnotationUserDefined的@MyAnnotation，反射可以获取注解信息  
class AnnotationUserDefinedExt extends AnnotationUserDefined{  
}  
```

在Java8之前，没有@ Repeatable注解时，要使用类似重复注解的功能必须要一个注解容器，这个容器也是注解类型的，还是上面那个@MyAnnotation为例，必须有一个@MyAnnotations这样的注解容器 

```java
@Inherited  
@Retention(RetentionPolicy.RUNTIME)  
@Target({ElementType.FIELD, ElementType.METHOD})  
@interface MyAnnotations{  
    MyAnnotation[] value();  
}  
```

而@MyAnnotation的声明则如下 

```java
@Inherited  
@Retention(RetentionPolicy.RUNTIME)  
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.TYPE})  
@interface MyAnnotation{  
    String value();  
}  
```

我们使用重复注解效果时，则如下 

```java
@MyAnnotations({  
        @MyAnnotation("默认的value数据赋值可以省略value = "),  
                @MyAnnotation(value = "不省略value = 得写法")})  
```

在Java8引入重复注解功能以后，@MyAnnotations注解容器的声明不变，我们只需要在@MyAnnotation注解的声明上加一行@Repeatable(MyAnnotations.class)就行了，如下 

```java
@Repeatable(MyAnnotations.class)  
@Inherited  
@Retention(RetentionPolicy.RUNTIME)  
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.TYPE})  
@interface MyAnnotation{  
    String value();  
}  
```

而我们在使用重复注解功能时，则如下 

```java
@MyAnnotation("默认的value数据赋值可以省略value = ")  
@MyAnnotation(value = "不省略value = 得写法")  
```

重复两个注解修饰同一个程序元素，就不需要吧注解容器写出来了。 

使用@Repeatable注解容器声明时修饰的元注解必须与该注解一样，@Target的枚举值不能只能少于该注解的范围，并且不能含有该注解没有的枚举值。如@MyAnnotations这个注解容器则不能使用ElementType.PARAMETER这个枚举值。 

### 提取注解信息 

提取注解信息是使用反射来实现的，反射只能获取@Retention(RetentionPolicy.RUNTIME)修饰的注解的信息，若是SOURCE和CLASS级别的注解，反射是获取不到的，只能使用代码分析框架和字节码分析框架是获取注解信息了。 

**Java的反射知识点参考我的博文[Java反射知识点总结]()**

Java的每种类型的程序元素（Class、Constructor、Field、Method、Parameter）都实现了AnnotatedElement接口，当反射获取到一个程序元素后，就可以使用以下方法获得该程序元素上修饰的注解和注解上的字段信息。 

```java
<T extends Annotation> T getAnnotation(Class<T> annotationClass) 
```

如果存在该元素的指定类型的注解，则返回这些注解，否则返回 null。 

```java
Annotation[] getAnnotations() 
```

返回此元素上存在的所有注解。 

```java
Annotation[] getDeclaredAnnotations() 
```

返回直接存在于此元素上的所有注解。 

```java
Boolean isAnnotationPresent(Class<? extends Annotation> annotationClass) 
```

如果指定类型的注解存在于此元素上，则返回 true，否则返回 false。 

Java8新增的获取注解的方法如下： 

```java
<T extends Annotation> T getDeclaredAnnotation(Class<T> annotationClass) 
```

如果存在直接存在于该元素的指定类型的注解，则返回这些注解，否则返回 null。 

```java
<T extends Annotation> T[] getAnnotationsByType(Class<T> annotationClass) 
```

由于Java8引入了重复注解，如果存在该元素的指定类型的注解，该方法则返回这些注解，否则返回 null。 

```java
<T extends Annotation> T[] getDeclaredAnnotationsByType(Class<T> annotationClass) 
```

由于Java8引入了重复注解，如果存在直接存在于该元素的指定类型的注解，该方法则返回这些注解，否则返回 null。 

还是刚刚的@MyAnnotation注解为例，我们使用反射获取该注解。 

```java
public static void main(String[] args) {  
        MyAnnotation anno = AnnotationUserDefined.class.getAnnotation(MyAnnotation.class);  
        MyAnnotation anno = AnnotationUserDefined.class.getAnnotation(MyAnnotation.class);  
        System.out.println(anno);     
        System.out.println(((Annotation) anno).annotationType());  
        System.out.println("该注解的value值：" + anno.value());   }  
```

代码执行结果如下 

![ReflectionGetAnnotationMethodValue](/images/Core_Java/JavaAnnotation/ReflectionGetAnnotationMethodValue.png)

AnnotatedElement接口的子接口AnnotatedType接口，还有AnnotatedType接口的四个子接口AnnotatedArrayType、AnnotatedParameterizedType、AnnotatedTypeVariable和AnnotatedWildcardType从名字上看跟Type接口及其四个子接口一一对应，似乎与泛型有关。文档中也没有有用的说明。 

```java
System.out.println(AnnotationUserDefined.class.getAnnotatedSuperclass());  
```

这一行代码将会打印出类似以下信息

```
sun.reflect.annotation.AnnotatedTypeFactory$AnnotatedTypeBaseImpl@6d06d69c 
```

sun开头的包名，说明AnnotatedTypeFactory是没有公开的API，反编译AnnotatedTypeFactory这个类，找到了分别找到了AnnotatedType接口的四个子接口AnnotatedArrayType、AnnotatedParameterizedType、AnnotatedTypeVariable和AnnotatedWildcardType的实现类。然后并没有找到什么有用的信息。。。 

注解上目前也是不支持泛型的使用。  
