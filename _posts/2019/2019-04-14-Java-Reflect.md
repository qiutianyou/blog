---
layout: post
title: Java反射机制
category: Java
tags: [Java]
---

认识Java反射机制，掌握常用反射对象的使用

## Class对象
前面一章ClassLoader学习中，我们知道类文件被类加载器加载到JVM中，会生成一个与之对应的一个Class类描述对象（即Class对象）
反射机制就是通过操作Class对象实现的。

Class对象功能：
Class对象描述了类的语义结构，可以从Class对象中获取构造函数、成员变量、方法等元素的反射对象，并 通过这些反射出来的对象对目标类对象进行操作。

## 主要反射类，以及获取反射类的方法
反射对象类定义在java.reflect包定义，通过java.lang.Class方法取得，其中最重要的三个反射类：

Constructor：

``` java

  /**
      * 返回所有public的构造函数
      */
  public Constructor<?>[] getConstructors() throws SecurityException {...}

  /**
      * 通过构造函数参数参数类型获取指定的public构造函数
      * 如果没有找到对应的构造函数,则抛出NoSuchMethodException
      */
  public Constructor<?>[] getConstructors() throws SecurityException {...}

  /**
      * 获取所有，包括非public类型的构造函数
      */
  public Constructor<?>[] getDeclaredConstructors() throws SecurityException {...}

  /**
      * 通过构造函数参数参数类型获取指定的构造函数（包括非public类型的）
      * 如果没有找到对应的构造函数,则抛出NoSuchMethodException
      */  
  public Constructor<?>[] getDeclaredConstructors() throws SecurityException {...}

```


Method：

``` java
// 获取所有方法对象
public Method[] getDeclaredMethods() throws SecurityException {
...
}

// 根据方法名、参数类型获取指定方法对象
public Method getDeclaredMethod(String name, Class<?>... parameterTypes)
...
}

...
//

```

Field：

``` java

// 获取所有属性对象对象
public Field[] getDeclaredFields() throws SecurityException {
  ...
}

// 获取指定属性对象
public Field getDeclaredField(String name)
      throws NoSuchFieldException, SecurityException {...}

```

## 反射类的使用
这里简单写个demo，让我们对反射有个直观的认识

``` java
public class Test {

    public static void main(String[] args) {

        try {

            // 根据全限类名获取class对象
            Class myTestClass = ClassLoader.getSystemClassLoader().loadClass("com.test.MyTest");

            // 通过class对象获取该类的有参构造函数
            Constructor constructor = myTestClass.getConstructor(String.class);
            // 设置为可访问
            constructor.setAccessible(true);
            // 通过构造函数生成实例
            Object ming = constructor.newInstance("小明");

            // 获取int类型的方法setAge
            Method setAge = myTestClass.getDeclaredMethod("setAge", int.class);
            // 设置为可访问
            setAge.setAccessible(true);
            // 调用方法
            setAge.invoke(ming, 20);

            // 获取指定的属性
            Field high = myTestClass.getDeclaredField("high");
            // 设置为可访问
            high.setAccessible(true);
            // 给ming这个对象的属性赋值
            high.set(ming, 180);


        } catch (Exception e) {
            System.out.println(e);
        }

    }


}

```

<b>如果需要对私有构造函数、方法、数，属性进行操作，则需要先设置为可访问，调用setAccessible(true)，</b>
