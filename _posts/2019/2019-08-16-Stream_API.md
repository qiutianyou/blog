---
layout: post
title: 常用的Stream操作
category: Stream
tags: [java]
---

为了更好地理解 Stream API，掌握一些常用的 Stream 操作十分必要

## collect(toList())

**collect(toList()) 方法由 Stream 里的值生成一个列表，是一个及早求值操作。**

Stream 的 of 方法使用一组初始值生成新的 Stream

例如：

``` java
List<String> collected = Stream.of("a", "b", "c")➊
.collect(Collectors.toList());➋

assertEquals(Arrays.asList("a", "b", "c"), collected);➌
```

这段程序展示了如何使用 collect(toList()) 方法从 Stream 中生成一个列表。如上文所述， 由于很多 Stream 操作都是惰性求值，因此调用 Stream 上一系列方法之后，还需要最后再 调用一个类似 collect 的及早求值方法。
这个例子也展示了本节中所有示例代码的通用格式。首先由列表生成一个 Stream ➊，然后 进行一些 Stream 上的操作，继而是 collect 操作，由 Stream 生成列表➋，最后使用断言 判断结果是否和预期一致➌。
形象一点儿的话，可以将 Stream 想象成汉堡，将最前和最后对 Stream 操作的方法想象成 两片面包，这两片面包帮助我们认清操作的起点和终点。

## map

**如果有一个函数可以将一种类型的值转换成另外一种类型，map 操作就可以 使用该函数，将一个流中的值转换成一个新的流。**


例如：

``` java
//使用 for 循环将字符串转换为大写
List<String> collected = new ArrayList<>();
for (String string : asList("a", "b", "hello")) {
          String uppercaseString = string.toUpperCase();
          collected.add(uppercaseString);
      }
assertEquals(asList("A", "B", "HELLO"), collected);
```

使用stream map实现

``` java
List<String> collected = Stream.of("a", "b", "hello").map(string -> string.toUpperCase()).collect(toList());

assertEquals(asList("A", "B", "HELLO"), collected);
```

传给 map 的 Lambda 表达式只接受一个 String 类型的参数，返回一个新的 String。参数 和返回值不必属于同一种类型，但是 Lambda 表达式必须是 Function 接口的一个实例，Function 接口是只包含一个参数的普通函数接口。

## filter

**遍历数据并检查其中的元素时，可尝试使用 Stream 中提供的新方法 filter**

例如：

``` java
//使用循环遍历列表，使用条件语句做判断，得到符合条件的列表
List<String> beginningWithNumbers = new ArrayList<>();
for(String value : asList("a", "1abc", "abc1")) {
  // isDigit是否是数字
  if (isDigit(value.charAt(0))) {
    beginningWithNumbers.add(value);
    }
}
assertEquals(asList("1abc"), beginningWithNumbers);
```

你可能已经写过很多类似的代码:这被称为 filter 模式，下面使用stream filter实现

``` java
List<String> beginningWithNumbers
      = Stream.of("a", "1abc", "abc1")
              .filter(value -> isDigit(value.charAt(0)))
              .collect(toList());

assertEquals(asList("1abc"), beginningWithNumbers);
```

和map很像，filter接受一个函数作为参数，该函数用Lambda表达式表示。该函数和前面 示例中 if 条件判断语句的功能一样，如果字符串首字母为数字，则返回 true。若要重构 遗留代码，for 循环中的 if 条件语句就是一个很强的信号，可用 filter 方法替代。


## flatMap

**flatMap 方法可用 Stream 替换值，然后将多个 Stream 连接成一个 Stream**

例如：

``` java
//包含多个列表的 Stream
List<Integer> together = Stream.of(asList(1, 2), asList(3, 4))
                              .flatMap(numbers -> numbers.stream())
                              .collect(toList());
assertEquals(asList(1, 2, 3, 4), together);
```

调用 stream 方法，将每个列表转换成 Stream 对象，其余部分由 flatMap 方法处理。 flatMap 方法的相关函数接口和 map 方法的一样，都是 Function 接口，只是方法的返回值 限定为 Stream 类型罢了。


## min/max

**Stream 上常用的操作之一是求最大值和最小值。Stream API 中的 max 和 min 操作足以解决 这一问题。**


``` java
/**
 * 文章实体类，
 *
 */
@Getter
@Setter
@AllArgsConstructor
class Essay {
  /** 文章名称 */
  private String essayName;
  /** 文章的阅读次数 */
  private int readCount;
}

/**
 * 测试
 */

// 初始化集合
List<Essay> essays = asList(new Essay("Essay1", 524),new Essay("Essay2", 378),new Track("Essay3", 451));

// min实现，需要传给它一个Comparator对象
Essay minEssay = essays.stream().min(Comparator.comparing(essay -> essay.getReadCount())).get();

assertEquals(essays.get(1), minEssay);
```

max/min方法需要传给它一个Comparator对象。Java 8提 供了一个新的静态方法 comparing，使用它可以方便地实现一个比较器。放在以前，我们需要比较两个对象的某项属性的值，现在只需要提供一个存取方法就够了。本例中使用 getLength 方法。
