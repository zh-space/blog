---
title: 微服务如何防止多次提交 study
date: 2022-1-18 
tags:
 - 微服务
---
## 今天突然想起了以前面试的时候有个面试官问我如何防止微服务多次提交，今天到网上学习了一下
## 前端拦截
点击按钮后把按钮变成不可点击状态，或者跳转另一个页面等方法都可以做到前端拦截，但是如果从后端直接调用请求呢。
## 后端拦截
后端拦截的实现思路是在方法执行之前，先判断此业务是否已经执行过，如果执行过则不再执行，否则就正常执行。
我们将请求的业务 ID 存储在内存中，并且通过添加互斥锁来保证多线程下的程序执行安全  
然而，将数据存储在内存中，最简单的方法就是使用 HashMap 存储，或者是使用 Guava Cache 也是同样的效果，但很显然 HashMap 可以更快的实现功能，所以我们先来实现一个 HashMap 的防重（防止重复）版本。
### 1.基础版——HashMap
```java
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
 
import java.util.HashMap;
import java.util.Map;
 
/**
 * 普通 Map 版本
 */
@RequestMapping("/user")
@RestController
public class UserController3 {
 
    // 缓存 ID 集合
    private Map<String, Integer> reqCache = new HashMap<>();
 
    @RequestMapping("/add")
    public String addUser(String id) {
        // 非空判断(忽略)...
        synchronized (this.getClass()) {
            // 重复请求判断
            if (reqCache.containsKey(id)) {
                // 重复请求
                System.out.println("请勿重复提交！！！" + id);
                return "执行失败";
            }
            // 存储请求 ID
            reqCache.put(id, 1);
        }
        // 业务代码...
        System.out.println("添加用户ID:" + id);
        return "执行成功！";
    }
}
```
存在的问题：此实现方式有一个致命的问题，因为 HashMap 是无限增长的，因此它会占用越来越多的内存，并且随着 HashMap 数量的增加查找的速度也会降低，所以我们需要实现一个可以自动“清除”过期数据的实现方案。
### 2.优化版——固定大小的数组
此版本解决了 HashMap 无限增长的问题，它使用数组加下标计数器（reqCacheCounter）的方式，实现了固定数组的循环存储。

当数组存储到最后一位时，将数组的存储下标设置 0，再从头开始存储数据，实现代码如下：
```java
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
 
import java.util.Arrays;
 
@RequestMapping("/user")
@RestController
public class UserController {
 
    private static String[] reqCache = new String[100]; // 请求 ID 存储集合
    private static Integer reqCacheCounter = 0; // 请求计数器（指示 ID 存储的位置）
 
    @RequestMapping("/add")
    public String addUser(String id) {
        // 非空判断(忽略)...
        synchronized (this.getClass()) {
            // 重复请求判断
            if (Arrays.asList(reqCache).contains(id)) {
                // 重复请求
                System.out.println("请勿重复提交！！！" + id);
                return "执行失败";
            }
            // 记录请求 ID
            if (reqCacheCounter >= reqCache.length) reqCacheCounter = 0; // 重置计数器
            reqCache[reqCacheCounter] = id; // 将 ID 保存到缓存
            reqCacheCounter++; // 下标往后移一位
        }
        // 业务代码...
        System.out.println("添加用户ID:" + id);
        return "执行成功！";
    }
}
```
### 3.扩展版——双重检测锁(DCL)
上一种实现方法将判断和添加业务，都放入 synchronized 中进行加锁操作，这样显然性能不是很高，于是我们可以使用单例中著名的 DCL（Double Checked Locking，双重检测锁）来优化代码的执行效率，实现代码如下：
```java
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
 
import java.util.Arrays;
 
@RequestMapping("/user")
@RestController
public class UserController {
 
    private static String[] reqCache = new String[100]; // 请求 ID 存储集合
    private static Integer reqCacheCounter = 0; // 请求计数器（指示 ID 存储的位置）
 
    @RequestMapping("/add")
    public String addUser(String id) {
        // 非空判断(忽略)...
        // 重复请求判断
        if (Arrays.asList(reqCache).contains(id)) {
            // 重复请求
            System.out.println("请勿重复提交！！！" + id);
            return "执行失败";
        }
        synchronized (this.getClass()) {
            // 双重检查锁（DCL,double checked locking）提高程序的执行效率
            if (Arrays.asList(reqCache).contains(id)) {
                // 重复请求
                System.out.println("请勿重复提交！！！" + id);
                return "执行失败";
            }
            // 记录请求 ID
            if (reqCacheCounter >= reqCache.length) reqCacheCounter = 0; // 重置计数器
            reqCache[reqCacheCounter] = id; // 将 ID 保存到缓存
            reqCacheCounter++; // 下标往后移一位
        }
        // 业务代码...
        System.out.println("添加用户ID:" + id);
        return "执行成功！";
    }
}
```
注意：DCL 适用于重复提交频繁比较高的业务场景，对于相反的业务场景下 DCL 并不适用。
### 4.完善版——LRUMap
上面的代码基本已经实现了重复数据的拦截，但显然不够简洁和优雅，比如下标计数器的声明和业务处理等，但值得庆幸的是 Apache 为我们提供了一个 commons-collections 的框架，里面有一个非常好用的数据结构 LRUMap 可以保存指定数量的固定的数据，并且它会按照 LRU 算法，帮你清除最不常用的数据。
小贴士：LRU 是 Least Recently Used 的缩写，即最近最少使用，是一种常用的数据淘汰算法，选择最近最久未使用的数据予以淘汰。
首先，我们先来添加 Apache commons collections 的引用：
```yaml
 <!-- 集合工具类 apache commons collections -->
<!-- https://mvnrepository.com/artifact/org.apache.commons/commons-collections4 -->
<dependency>
  <groupId>org.apache.commons</groupId>
  <artifactId>commons-collections4</artifactId>
  <version>4.4</version>
</dependency>
```
实现代码如下：
```java
import org.apache.commons.collections4.map.LRUMap;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
 
@RequestMapping("/user")
@RestController
public class UserController {
 
    // 最大容量 100 个，根据 LRU 算法淘汰数据的 Map 集合
    private LRUMap<String, Integer> reqCache = new LRUMap<>(100);
 
    @RequestMapping("/add")
    public String addUser(String id) {
        // 非空判断(忽略)...
        synchronized (this.getClass()) {
            // 重复请求判断
            if (reqCache.containsKey(id)) {
                // 重复请求
                System.out.println("请勿重复提交！！！" + id);
                return "执行失败";
            }
            // 存储请求 ID
            reqCache.put(id, 1);
        }
        // 业务代码...
        System.out.println("添加用户ID:" + id);
        return "执行成功！";
    }
}
```
使用了 LRUMap 之后，代码显然简洁了很多。
### 5.最终版——封装
以上都是方法级别的实现方案，然而在实际的业务中，我们可能有很多的方法都需要防重，那么接下来我们就来封装一个公共的方法，以供所有类使用：
```java
import org.apache.commons.collections4.map.LRUMap;
 
/**
 * 幂等性判断
 */
public class IdempotentUtils {
 
    // 根据 LRU(Least Recently Used，最近最少使用)算法淘汰数据的 Map 集合，最大容量 100 个
    private static LRUMap<String, Integer> reqCache = new LRUMap<>(100);
 
    /**
     * 幂等性判断
     * @return
     */
    public static boolean judge(String id, Object lockClass) {
        synchronized (lockClass) {
            // 重复请求判断
            if (reqCache.containsKey(id)) {
                // 重复请求
                System.out.println("请勿重复提交！！！" + id);
                return false;
            }
            // 非重复请求，存储请求 ID
            reqCache.put(id, 1);
        }
        return true;
    }
}
```
调用代码如下
```java
import com.example.idempote.util.IdempotentUtils;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
 
@RequestMapping("/user")
@RestController
public class UserController4 {
    @RequestMapping("/add")
    public String addUser(String id) {
        // 非空判断(忽略)...
        // -------------- 幂等性调用（开始） --------------
        if (!IdempotentUtils.judge(id, this.getClass())) {
            return "执行失败";
        }
        // -------------- 幂等性调用（结束） --------------
        // 业务代码...
        System.out.println("添加用户ID:" + id);
        return "执行成功！";
    }
}
```
小贴士：一般情况下代码写到这里就结束了，但想要更简洁也是可以实现的，你可以通过自定义注解，将业务代码写到注解中，需要调用的方法只需要写一行注解就可以防止数据重复提交了，老铁们可以自行尝试一下
### 扩展知识——LRUMap 实现原理分析
既然 LRUMap 如此强大，我们就来看看它是如何实现的。
LRUMap 的本质是持有头结点的环回双链表结构，它的存储结构如下：
```java
AbstractLinkedMap.LinkEntry entry;
```
当调用查询方法时，会将使用的元素放在双链表 header 的前一个位置，源码如下：
```java
public V get(Object key, boolean updateToMRU) {
    LinkEntry<K, V> entry = this.getEntry(key);
    if (entry == null) {
        return null;
    } else {
        if (updateToMRU) {
            this.moveToMRU(entry);
        }
 
        return entry.getValue();
    }
}
protected void moveToMRU(LinkEntry<K, V> entry) {
    if (entry.after != this.header) {
        ++this.modCount;
        if (entry.before == null) {
            throw new IllegalStateException("Entry.before is null. This should not occur if your keys are immutable, and you have used synchronization properly.");
        }
 
        entry.before.after = entry.after;
        entry.after.before = entry.before;
        entry.after = this.header;
        entry.before = this.header.before;
        this.header.before.after = entry;
        this.header.before = entry;
    } else if (entry == this.header) {
        throw new IllegalStateException("Can't move header to MRU This should not occur if your keys are immutable, and you have used synchronization properly.");
    }
 
}
```
如果新增元素时，容量满了就会移除 header 的后一个元素，添加源码如下：
```java
protected void addMapping(int hashIndex, int hashCode, K key, V value) {
     // 判断容器是否已满 
     if (this.isFull()) {
         LinkEntry<K, V> reuse = this.header.after;
         boolean removeLRUEntry = false;
         if (!this.scanUntilRemovable) {
             removeLRUEntry = this.removeLRU(reuse);
         } else {
             while(reuse != this.header && reuse != null) {
                 if (this.removeLRU(reuse)) {
                     removeLRUEntry = true;
                     break;
                 }
                 reuse = reuse.after;
             }
             if (reuse == null) {
                 throw new IllegalStateException("Entry.after=null, header.after=" + this.header.after + " header.before=" + this.header.before + " key=" + key + " value=" + value + " size=" + this.size + " maxSize=" + this.maxSize + " This should not occur if your keys are immutable, and you have used synchronization properly.");
             }
         }
         if (removeLRUEntry) {
             if (reuse == null) {
                 throw new IllegalStateException("reuse=null, header.after=" + this.header.after + " header.before=" + this.header.before + " key=" + key + " value=" + value + " size=" + this.size + " maxSize=" + this.maxSize + " This should not occur if your keys are immutable, and you have used synchronization properly.");
             }
             this.reuseMapping(reuse, hashIndex, hashCode, key, value);
         } else {
             super.addMapping(hashIndex, hashCode, key, value);
         }
     } else {
         super.addMapping(hashIndex, hashCode, key, value);
     }
 }
```
判断容量的源码：
```java
public boolean isFull() {
  return size >= maxSize;
}
```
容量未满就直接添加数据：
```java
super.addMapping(hashIndex, hashCode, key, value);
```
如果容量满了，就调用 reuseMapping 方法使用 LRU 算法对数据进行清除。
综合来说：LRUMap 的本质是持有头结点的环回双链表结构，当使用元素时，就将该元素放在双链表 header 的前一个位置，在新增元素时，如果容量满了就会移除 header 的后一个元素。

