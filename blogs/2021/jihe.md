---
title: 集合 study
date: 2021-12-22 
tags:
 - java
---
## 今天整理了一下自己以前学习集合的笔记
```java
package com.example;

import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import javax.annotation.processing.Filer;
import java.io.*;
import java.util.*;

@SpringBootTest
class SpringsecurityApplicationTests {
    @Test
    void contextLoads() {

        String d="123";
        int e=Integer.parseInt(d);
        String a="abcde";
        char[] c=a.toCharArray();
        String[] b=a.split("");
        for (int i = 0; i < c.length; i++) {
           System.out.println( c[i]);
        }
    }
    @Test
    void test(){
        List giao=new ArrayList();
        Collection collection=new ArrayList();
        collection.add("苹果");
        collection.add("西瓜");
        collection.add("榴莲");
        //System.out.println(collection.size());
        //System.out.println(collection);
        //删除
        //collection.remove("苹果");
        //System.out.println("删除之后:"+collection.size());
        //遍历 collection不能用for遍历
        //for(Object item:collection){
            //System.out.println(item);
       //}
        //迭代器，专门用来遍历集合的接口
        Iterator iterator = collection.iterator();
        while (iterator.hasNext()){
            String giaoo=(String)iterator.next();
            System.out.println(giaoo.toString());
        }
        System.out.println(collection.size());
    }
    @Test
    public void test1(){
        Collection collection=new ArrayList();
        User user=new User(10,"张三");
        User user1=new User(11,"李四");
        //添加元素
        collection.add(user);
        collection.add(user1);
        //删除
        collection.remove(user);
        System.out.println(collection);
    }
    @Test
    public void test2(){
        List<Integer> a=new ArrayList<Integer>();
        a.add(1);
        a.add(2);
        a.add(0,3);
        a.remove((Object)3);
        //for (int i = 0; i < a.size(); i++) {
            //System.out.println(a.get(i));
        //}
        //for(Object o:a){
            //System.out.println(o);
        //}
        //Iterator iterator = a.iterator();
        //while (iterator.hasNext()){
            //System.out.println(iterator.next());
        //}
        ListIterator listIterator = a.listIterator();
        while (listIterator.hasNext()){
            System.out.println(listIterator.nextIndex()+":"+listIterator.next());
        }
        while (listIterator.hasPrevious()){
            System.out.println(listIterator.previousIndex()+":"+listIterator.previous());
        }
    }
    @Test
    public void test3(){
        ArrayList giao=new ArrayList<>();
        User user=new User(10,"张三");
        User user1=new User(11,"李四");
        giao.add(user);
        giao.add(user1);
//        System.out.println(giao.size());
//        System.out.println(giao.toString());
//        giao.remove(new User(10,"张三"));
//        System.out.println(giao.size());
//        System.out.println(giao.toString());
//        Iterator iterator = giao.iterator();
//        while (iterator.hasNext()){
//            System.out.println(iterator.next());
//        }
        ListIterator listIterator = giao.listIterator();
        while (listIterator.hasPrevious()){
            String s= (String) listIterator.previous();
            System.out.println(s.toString());
        }
    }
    @Test
    public void test4(){
        //创建集合
        LinkedList linkedList=new LinkedList();
        //1.添加元素
        User user=new User(10,"张三");
        User user1=new User(11,"李四");
        User user2=new User(12,"王五");
        linkedList.add(user);
        linkedList.add(user1);
        linkedList.add(user2);
        //System.out.println(linkedList.size());
        //System.out.println(linkedList);
        //2.删除
        //.remove(new User(10,"张三"));
        //System.out.println(linkedList.size());
        //3.遍历
//        for (Object o:linkedList
//             ) {
//            System.out.println(o);
//        }
//        ListIterator listIterator = linkedList.listIterator();
//        while (listIterator.hasPrevious()){
//            listIterator.previous();
//        }
    }
    @Test
    public void test5(){
        MyGeneric<String> myGeneric=new MyGeneric<>();
        myGeneric.t="泛型";
        myGeneric.giao("大家好");
        String t = myGeneric.getT();
        System.out.println(t);
        MyGeneric<Integer> myGeneric1=new MyGeneric<>();
        myGeneric1.t=2;
        myGeneric1.giao(1);
        Integer integer= myGeneric1.getT();
        System.out.println(integer);
    }
    @Test
    public void test6(){
        //MyGenericInterface<String> myGenericInterface=new MyInterfaceImp<String>();
        //myGenericInterface.server("泛型接口");
//        User user=new User();
//        user.show("我giao");
        ArrayList<String> arrayList=new ArrayList<String>();
        arrayList.add("a");
        //arrayList.add(1);
        for (Object o:arrayList
             ) {
            System.out.println(o);
        }
    }
    @Test
    public void test7(){
        Set<String> set=new HashSet<>();
        //添加数据
        set.add("a");
        set.add("b");
        set.add("c");
        set.add("d");
        //System.out.println("数据个数:"+set.size());
        //System.out.println("数据个数:"+set.toString());
//        for (String s:set
//             ) {
//            System.out.println(s);
//        }
        Iterator<String> iterator = set.iterator();
        while (iterator.hasNext()){
            String ss=iterator.next();
            System.out.println(ss);
        }
    }
    @Test
    public void test8(){
        HashSet<String> hashSet=new HashSet<String>();
        hashSet.add("a");
        hashSet.add("b");
        hashSet.add("c");
        hashSet.add("d");
        Iterator<String> iterator = hashSet.iterator();
        while (iterator.hasNext()){
            System.out.println(iterator.next());
        }
    }
    @Test
    public void test9(){
        HashSet<User> hashSet=new HashSet();
        User user=new User(10,"a");
        User user1=new User(20,"b");
        User user2=new User(30,"c");
        User user3=new User(40,"d");
        hashSet.add(user);
        hashSet.add(user1);
        hashSet.add(user2);
        hashSet.add(user3);
        hashSet.add(new User(40,"d"));
        for (User u:hashSet
             ) {
            System.out.println(u);
        }
    }
    @Test
    public void test10(){
        TreeSet<User> tree=new TreeSet<>();
        User user=new User(10,"a");
        User user1=new User(110,"b");
        User user2=new User(30,"c");
        User user3=new User(40,"d");
        tree.add(user);
        tree.add(user1);
        tree.add(user2);
        tree.add(user3);
        System.out.println(tree.size());
        System.out.println(tree.toString());
    }
    @Test
    public void test11(){
        TreeSet treeSet=new TreeSet(new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                int n1=o1.length()-o2.length();
                int n2=o1.compareTo(o2);
                return n1==0?n2:n1;
            }
        });
        treeSet.add("asdasd");
        treeSet.add("sfdhgdfh");
        treeSet.add("afdhgdfh");
        treeSet.add("asgsdasasfrhgd");
        treeSet.add("asdasfghdfsd");
        System.out.println(treeSet);
    }
    @Test
    public void test12(){
        Map<String,String> map=new HashMap<>();
        map.put("a","b");
        map.put("c","d");
        map.put("e","f");
        //System.out.println(map.size());
        //System.out.println(map.keySet());
//        for (String s:map.keySet()
//             ) {
//            System.out.println("key:"+s+"value:"+map.get(s));
//        }
        //System.out.println(map.entrySet());
//        Set<Map.Entry<String,String>> entries=map.entrySet();
//        System.out.println(entries);
//        for (Map.Entry<String,String> entry:entries
//             ) {
//            System.out.println(entry.getKey()+"**"+entry.getValue());
//        }
    }
    @Test
    public void test13(){
        List<String> list=new ArrayList();
        list.add("asdasd");
        list.add("xdvszxdv");
        list.add("asdfa");
        list.add("tyjktyghksd");
        list.add("asdgfv");
        Collections.sort(list, new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                int n1=o1.length()-o2.length();
                int n2=o1.compareTo(o2);
                return n1==0?n2:n1;
            }
        });
        Collections.reverse(list);

        //int asdfa = Collections.binarySearch(list, "tyjktyghksd");
        Collections.shuffle(list);
        System.out.println(list);
        //list转为数组
        String[] strings = list.toArray(new String[0]);
        for (int i=0;i<strings.length;i++){
            System.out.println(strings[i]);
        }
        System.out.println(Arrays.toString(strings));
        //System.out.println(asdfa);
        //数组转集合\
        String[] names={"asdasd","dfgsg"};
        //受限集合，不能添加和删除
        List<String> list1 = Arrays.asList(names);
        Integer[] giao={10,20,30};
        List<Integer> ints = Arrays.asList(giao);
    }
    @Test
    public void test14(){
        int[] arr = {1,4,1,4,2,5,4,5,8,7,8,77,88,5,4,9,6,2,4,1,5};
        int a=0;
        for (int i = 0; i < arr.length; i++) {
            String s1 = String.valueOf(arr[i]);
            for (int j = 0; j < arr.length; j++) {
                String s = String.valueOf(arr[j]);
                if (arr[i]==arr[j] || s.contains(s1)){
                    a=a+1;
                }
            }
            System.out.println(arr[i]+"出现:"+a);
            a=0;
        }
    }
    @Test
    public void test15(){
        for (int i = 2; i < 100; i++) {
            for (int j = 2; j <=i; j++) {
                if (i%j==0){
                   if(i==j){
                       System.out.println(i);
                   }else{
                       break;
                   }
                }
            }
        }
    }
    @Test
    public void test16(){
        ArrayList<String> a=new ArrayList();
        ArrayList<String> b=new ArrayList();
        ArrayList<String> c=new ArrayList();
        //添加队列元素
        a.add("a");
        a.add("b");
        a.add("c");
        a.add("d");
        //进堆
        for (String s:a
             ) {
            c.add(s);
        }
        //出堆
        for (String s:c){
            System.out.print("出堆"+s);
        }
        //进栈
        for (String s:a){
            b.add(s);
        }
        //出栈
        Collections.reverse(b);
        for (String s:b){
            System.out.print("出栈"+s);
        }
    }
    @Test
    public void test17(){
        Map<String,String> map=new HashMap<>();
        map.put("","");
        map.put("","z");
        Set<Map.Entry<String,String>>set=map.entrySet();
        for (Map.Entry<String,String> mapp:set
             ) {
            System.out.println(mapp);
        }
    }
    @Test
    public void test18(){
        System.out.println("123".valueOf("string"));
    }
}

```
