---
layout: post
title: 类加载器 —— ClassLoader
category: Java
tags: [Java]
---
了解类加载器的工作机制，Class描述对象，重要方法和日常问题。了解Class描述对象与Class实例对象的关系

## 类加载器的工作机制
类加载器就是寻找类的字节码文件并构造出类在JVM内部表示的对象组件。在Java中，类加载器把一个类装入JVM中，要经过以下步骤：
1. 加载：寻找和导入class文件
2. 链接：执行校验、准备和解析步骤
    a. 校验：检查载入的class文件的数据正确性
    b. 准备：给类的静态变量分配存储空间
    c. 解析：将符号引用转成直接引用
3. 初始化：对类的静态变量、静态代码块执行初始化工作

内存分配不清楚没关系，接下来会更新JVM内存模型

类加载器工作是由ClassLoader及其子类完成的，ClassLoader是一个重要的Java运行时系统组件，它负责运行时查找和装入class字节码文件。

JVM运行时会产生三个ClassLoader，分别是：根加载器、ExtClassLoader、AppClassLoader，其中根加载器不是ClassLoader的子类，根加载器是由C++编写的。
根加载器：负责加载 jre的核心类库
ExtClassLoader：负责加载jre扩展目录的类包
AppClassLoade：负责加载ClassPath路径下的类包

这三个类加载器是存在父子层级关系的：
根加载器是ExtClassLoader的父加载器；
ExtClassLoader是AppClassLoader的父加载器

通过getParent()可以获得父加载器对象（需要注意的是，根加载器是由c++编写的，Java中无法获取它的句柄，所以会返回null）


## Class描述对象
类文件被加载并解析后，在JVM内将有一个对应的java.lang.Class类描述对象，该类的实例都有指向这个类描述对象的引用，而类描述对象又拥有指向关联的ClassLoader的引用，可以知道是由哪个类加载器加载的，因为除了JVM默认提供的三个ClassLoader以外，还可以编写第三方类加载器

附一张手机拍的关系图
![classLoader](/assets/images/ClassLoader.png)

## 重要方法

``` java
public abstract class ClassLoader {

  /**
     * 获取父级类加载器，所有类加载器有且仅有一个父类加载器
     */
  private final ClassLoader parent;


  /**
     * 参数name是全限类名，例如：com.test.Test。
     * 它有一个带boolean值重载方法，控制是否需要对其解析。如果jvm只需要知道该类是否存在或者找到该类的超类，则不需要对其解析
     */
  public Class<?> loadClass(String name) throws ClassNotFoundException {
          return loadClass(name, false);
      }

  /**
     * 将类文件的字节数组转化为JVM内部的Class对象
     * 字节数组可以是本地的，也可以是远程获取的，name是全限类名
     */
  protected final Class<?> defineClass(String name, byte[] b, int off, int len)
        throws ClassFormatError
    {
        return defineClass(name, b, off, len, null);
    }

  /**
     * 从本地文件系统中载入class文件
     * 如果本地不存在该class文件，则抛出ClassNotFoundException异常
     * 该方法是JVM默认使用的加载机制
     */
  protected final Class<?> findSystemClass(String name)
        throws ClassNotFoundException
    {
        ClassLoader system = getSystemClassLoader();
        if (system == null) {
            if (!checkName(name))
                throw new ClassNotFoundException(name);
            Class<?> cls = findBootstrapClass(name);
            if (cls == null) {
                throw new ClassNotFoundException(name);
            }
            return cls;
        }
        return system.loadClass(name);
    }

  /**
     * 调用该方法来查看ClassLoader是否已装入某个类，如果已装入则返回Class，否则返回null
     */
  protected final Class<?> findLoadedClass(String name) {
      if (!checkName(name))
          return null;
      return findLoadedClass0(name);
  }
}
```

## 日常问题
java.lang.NoSuchMethodError
  我们开发时使用第三方jar包时，编译能通过，运行时有可能报这个错误，这基本就是类加载时出现的冲突问题，例如：同时引用了abc-1.0.jar、abc-2.0.jar，使用的方法恰好是2.0有，1.0没有，JVM碰巧从1.0的类包里加载类，就会报NoSuchMethodError。

  修复&排查方法：
    1. 版本冲突，查看引用的类包各版本方法；
    2. 情况比较复杂，引用的类包很庞大，那么就需要，通过工具查看JVM引用类是从哪个类包加载的
