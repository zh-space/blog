---
title: 流式编程和lambda表达式 study
date: 2021-12-17 
tags:
 - java
 - 流式编程和lambda表达式
---
## 流式编程
### 创建流
```java
//产生从0到200的随机浮点数的流
DoubleStream randomStream = new Random().doubles(0, 200);

//将数组转换成流，可以产生基本数据类型的流，
//如IntStream、FloatStream、DoubleStream等，运行效率比较高
DoubleStream arrayStream= Arrays.stream(new double[]{1,3,4,5,2,11});

//根据一组对象产生流，但不能产生基本数据类型的流，只能产生对应包装类的流
Stream<String> stream = Stream.of("happy", "sad", "bad", "yes", "no");

//产生从0到9的整数流
IntStream rangeStream=IntStream.range(0,10);

//以第一个参数为种子，迭代产生后面对象的流
Stream<Integer> iterateStream = Stream.iterate(0, i->2*i);

//大部分集合都有stream()方法用来产对应的流
Stream<String> collectionStream = new ArrayList<String>().stream();
Stream<String> parallelStream = new ArrayList<String>().parallelStream();
//通过parallelStream()方法可以产生一个并行流，Java会将操作在多个核心上运行
```
### 中间操作
中间操作具体包括去重、过滤、映射等操作，值得说明的是，在执行中间操作的代码的时候并不会执行这些操作，而只会把这些操作保存在流里面，每次中间操作都会产生一个新的流对象，保存从开始到现在要进行的所有操作序列，在执行结束操作的时候才会真正执行这些操作，这叫做懒加载。
```java
peek() 操作的目的是帮助调试。它允许你无修改地查看流中的元素。代码示例：
// streams/Peeking.java
class Peeking {
    public static void main(String[] args) throws Exception {
        Stream.of("Will I eat the apple?".split(" "))
        .map(w -> w + " ")
        .peek(System.out::print)
        .map(String::toUpperCase)
        .peek(System.out::print)
        .map(String::toLowerCase)
        .forEach(System.out::print);
    }
}
输出结果：
Will WILL will I I i eat EAT eat the THE the apple APPLE apple
```
### 流元素排序
在最开始的代码中，我们熟识了 sorted() 的无参数方法。其实它还有另一种形式的实现：传入一个 Comparator 参数。代码示例：
```java
// streams/SortedComparator.java
import java.util.*;
public class SortedComparator {
    public static void main(String[] args) throws Exception {
        //PS:这里的FileToWords是一个可以将文本文件转换成单词流的类
        FileToWords.stream("Cheese.dat")
        .skip(10)
        .limit(10)
        .sorted(Comparator.reverseOrder())
        .map(w -> w + " ")
        .forEach(System.out::print);
    }
}
输出结果：
you what to the that sir leads in district And

sorted() 预设了一些默认的比较器。这里我们使用的是反转“自然排序”。你当然也可以把 Lambda 函数作为参数传递给 sorted()。
```
### 移除元素
```java
distinct()：最开始的代码中的 distinct() 可用于消除流中的重复元素。相比创建一个 Set 集合，该方法的效率要高很多。
filter(Predicate)：过滤操作则会留下使过滤器方法返回值为 true的元素。。
在下例中，isPrime() 作为过滤器函数，用于检测质数。
import java.util.stream.*;

public class Prime {
    public static Boolean isPrime(long n) {
        return LongStream.rangeClosed(2, (long)Math.sqrt(n))
            .noneMatch(i -> n % i == 0);
            //如果流中所有元素调用上述方法都返回false，则noneMatch()返回true
    }
    
    public LongStream numbers() {
        return LongStream.iterate(2, i -> i + 1)
            .filter(Prime::isPrime);
    }
    
    public static void main(String[] args) {
        new Prime().numbers()
            .limit(10)
            .forEach(n -> System.out.format("%d ", n));
        System.out.println();
        new Prime().numbers()
            .skip(90)
            .limit(10)
            .forEach(n -> System.out.format("%d ", n));
    }
}

输出结果：
2 3 5 7 11 13 17 19 23 29
467 479 487 491 499 503 509 521 523 541

与range()是左闭右开区间不同，rangeClosed() 是闭区间，左右的值都包括。如果不能整除，即余数不等于 0，则 noneMatch() 操作返回 true，如果出现任何等于 0 的结果则返回 false。
```
### 应用函数到元素
```java
map(Function)：将原来流中的每个元素都调用参数里的方法，其返回值汇总起来产生一个新的流。
mapToInt(ToIntFunction)：操作同上，但结果是IntStream。
mapToLong(ToLongFunction)：操作同上，但结果是 LongStream。
mapToDouble(ToDoubleFunction)：操作同上，但结果是 DoubleStream。

之前的代码中就多次用到了map方法，只需要知道它可以将流里的所有元素都变成与其对应的新元素就可以了，这里就不进行代码展示了。

最后需要注意的一点是，同一个流只能进行一次操作，例如下面的代码就会报错：
```
```java
//以第一个参数为种子，迭代产生后面对象的流
Stream<Integer> iterateStream= Stream.iterate(0, i->2*i).limit(10);
iterateStream.map(Integer::doubleValue);
iterateStream.map(Integer::byteValue);  //这句语句会报错
```
### 结束操作
这些操作接收一个流并产生一个最终结果；它们不会向后面的流提供任何东西。因此，结束操作总是你在管道中做的最后一个操作。

转化为数组
```java
toArray()：将流转换成适当类型的数组。
toArray(generator)：在特殊情况下，生成器用于分配自定义的数组存储
```
遍历
```java
forEach(Consumer)：你已经看到过很多次 System.out::println 作为 Consumer 函数。
forEachOrdered(Consumer)： 确保按照原始流的顺序执行。
```
### 收集
```java
collect(Collector)：使用 Collector 收集流元素到结果集合中。
collect(Supplier, BiConsumer, BiConsumer)：收集流元素到结果集合中，第一个参数用于创建一个新的结果集合，第二个参数用于将下一个元素加入到现有结果合集中，第三个参数用于将两个结果合集合并。
```
第一种形式中的的Collector参数，Java核心库为我们提供了很多Collector实现类，都在Collectors这个工具类里面，例如Colletors.toList、Collectos.toMap、Collections.toCollection等等，基本上都是故名思义，就不介绍了。当然也可以自己写写一个类来继承自Collector来实现自定义的收集要求

但对于自定义的元素收集要求，最好的办法还是采用第二种形式，让我们来看一个示例代码：
```java
import java.util.*;
import java.util.stream.*;
public class SpecialCollector {
    public static void main(String[] args) throws Exception {
        ArrayList<String> words = FileToWords.stream("Cheese.dat")
            .collect(ArrayList::new, ArrayList::add, ArrayList::addAll);
        words.stream()
            .filter(s -> s.equals("cheese"))
            .forEach(System.out::println);
    }
}
```
匹配
```java
allMatch(Predicate) ：如果流的每个元素根据提供的 Predicate 都返回 true 时，最终结果返回为 true。这个操作将会在第一个 false 之后短路，也就是不会在发生 false 之后继续执行计算。
anyMatch(Predicate)：如果流中的任意一个元素根据提供的 Predicate 返回 true 时，最终结果返回为 true。这个操作将会在第一个 true 之后短路，也就是不会在发生 true 之后继续执行计算。
noneMatch(Predicate)：如果流的每个元素根据提供的 Predicate 都返回 false 时，最终结果返回为 true。这个操作将会在第一个 true 之后短路，也就是不会在发生 true 之后继续执行计算。
```
元素查找
```java
findFirst()：返回一个含有第一个流元素的 Optional类型的对象，如果流为空返回 Optional.empty。
findAny()：返回含有任意流元素的 Optional类型的对象，如果流为空返回 Optional.empty
```
findFirst() 无论流是否为并行化的，总是会选择流中的第一个元素。对于非并行流，findAny()会选择流中的第一个元素（但从定义上来看是选择任意元素）。

统计信息
```java
average() ：求取流元素平均值。
max() 和 min()：求元素的最大值和最小值，对于非数字流则要多一个Comparator参数。
sum()：对所有流元素进行求和。
count()：流中的元素个数。
```
## lambda表达式
### 要点 1：lambda 表达式的使用位置
预定义使用了 @Functional 注释的函数式接口，自带一个抽象函数的方法，或者 SAM（Single Abstract Method 
单个抽象方法）类型。这些称为 lambda 表达式的目标类型，可以用作返回类型，或 lambda 目标代码的参数。例如，
若一个方法接收 Runnable、Comparable 或者 Callable 接口，都有单个抽象方法，可以传入 lambda 表达式。类似
的，如果一个方法接受声明于 java.util.function 包内的接口，例如 Predicate、Function、Consumer 或 Supplier，
那么可以向其传 lambda 表达式。
### 要点 2：lambda 表达式和方法引用
lambda 表达式内可以使用方法引用，仅当该方法不修改 lambda 表达式提供的参数。本例中的 lambda 表达式
可以换为方法引用，因为这仅是一个参数相同的简单方法调用。
```java
list.forEach(n -> System.out.println(n));
list.forEach(System.out::println); // 使用方法引用
```
然而，若对参数有任何修改，则不能使用方法引用，而需键入完整地 lambda 表达式，如下所示：
```java
list.forEach((String s) -> System.out.println("*" + s + "*"));
```
事实上，可以省略这里的 lambda 参数的类型声明，编译器可以从列表的类属性推测出来。
### 要点 3：lambda 表达式内部引用资源
lambda 内部可以使用静态、非静态和局部变量，这称为 lambda 内的变量捕获。
### 要点 4：lambda 表达式也成闭包
Lambda 表达式在 Java 中又称为闭包或匿名函数，所以如果有同事把它叫闭包的时候，不用惊讶。
### 要点 5：lambda 表达式的编译方式
Lambda 方法在编译器内部被翻译成私有方法，并派发 invokedynamic 字节码指令来进行调用。可以使用 JDK
中的 javap 工具来反编译 class 文件。使用 javap -p 或 javap -c -v 命令来看一看 lambda 表达式生成的字节码。
大致应该长这样：
```java
private static java.lang.Object lambda$0(java.lang.String);
```
### 要点 6：lambda 表达式的限制
lambda 表达式有个限制，那就是只能引用 final 或 final 局部变量，这就是说不能在 lambda 内部修改定义在
域外的变量。
```java
List<Integer> primes = Arrays.asList(new Integer[]{2, 3,5,7});
int factor = 2;
primes.forEach(element -> { factor++; });
Error：
Compile time error : "local variables referenced from a lambda expression must be final or 
effectively final"
```
另外，只是访问它而不作修改是可以的，如下所示：
```java
List<Integer> primes = Arrays.asList(new Integer[]{2, 3,5,7});
int factor = 2;
primes.forEach(element -> { System.out.println(factor*element); });
输出：
4
6
10
14
```
因此，它看起来更像不可变闭包，类似于 Python。
### 
```java
package com.example;

import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.ArrayList;
import java.util.List;

@SpringBootTest
class Testt {
    @Test
    public void test1(){
        List list=new ArrayList();
        Integer z=1;
        list.add(1);
        list.add("2");
        list.add(3);
        list.add("4");
        list.forEach(n->{
            System.out.println(n);
            System.out.println(n+"z");
        });
        list.forEach((n)-> System.out.println("*"+n));
        list.forEach(System.out::println);
        //lambda 表达式有个限制，那就是只能引用 final 或 final 局部变量，这就是说不能在 lambda 内部修改定义在域外的变量。
        //list.forEach(n->System.out.println(z++));
        //另外，只是访问它而不作修改是可以的，如下所示：
        list.forEach(n->System.out.println(z));
    }
}

```
