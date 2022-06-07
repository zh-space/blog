---
title: 线程 study
date: 2021-12-22 
tags:
 - java
 - 线程
---
## 今天整理了一下自己以前学习线程的笔记
```java
//创建线程方法一  继承Tread类，重写run方法，调用start方法
public class TestThread extends Thread{
    @Override
    public void run() {
        //run方法线程体
        for (int i=0;i<20;i++){
            System.out.println("我在学习线程"+i);
        }
        super.run();
    }

    public static void main(String[] args) {
        TestThread testThread=new TestThread();
        testThread.start();
        //main线程，主线程
        for (int i=0;i<20;i++){
            System.out.println("我在学习主线程"+i);
        }
    }
}
```
```java
public class TestThread1 implements Runnable{
    @Override
    public void run() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("我在学习线程");
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService pool= Executors.newFixedThreadPool(2);
        TestThread1 thread11=new TestThread1();
        Thread thread=new Thread(thread11);
        Thread thread1=new Thread(thread11);
        Thread thread2=new Thread(thread11);
        Thread thread3=new Thread(thread11);
        Thread thread4=new Thread(thread11);
        Thread thread5=new Thread(thread11);
        Thread thread6=new Thread(thread11);
        Thread thread7=new Thread(thread11);
        pool.execute(thread);
        pool.execute(thread1);
        pool.execute(thread2);
        pool.execute(thread3);
        pool.execute(thread4);
        pool.execute(thread5);
        pool.execute(thread6);
        Future<?> submit = pool.submit(thread7);
        pool.shutdown();
        System.out.println(submit.get());
    }
}
```
```java
class TestThread2 implements Runnable{
    ArrayList arrayList=new ArrayList();
    @Override
    public void run() {
        int i=1;
        //synchronized (arrayList){
            arrayList.add(i);
            System.out.println("存放数量:"+arrayList.size());
        //}
//        try {
//            Thread.sleep(1000);
//        } catch (InterruptedException e) {
//            e.printStackTrace();
//        }
    }

    public static void main(String[] args) {
        TestThread2 testThread2=new TestThread2();
        for (int i = 0; i < 1000; i++) {
            Thread thread=new Thread(testThread2);
            thread.start();
        }
    }
}
```
## 龟兔赛跑
```java
public class TestYread3 implements Runnable{
    private static String winner;
    @Override
    public void run() {

        //胜利者
        for (int i=0;i<=100;i++){
            if (Thread.currentThread().getName().equals("兔子")&&i%10==0){
                try {
                    Thread.sleep(200);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            boolean tags=gameocer(i);
            if (tags){
                break;
            };
            System.out.println(Thread.currentThread().getName()+"跑了------>"+i+"步");
        }
    }
    //判断是否完成比赛
    private synchronized boolean gameocer(int steps){
        if (winner!=null){
            return false;
        }else{
            if (steps>=100){
                winner=Thread.currentThread().getName();
                System.out.println("winer is"+winner);
                return true;
            }
        }
        return false;
    }

    public static void main(String[] args) {
        TestYread3 testYread3=new TestYread3();
        Thread thread=new Thread(testYread3,"乌龟");
        Thread thread1=new Thread(testYread3,"兔子");
        thread.start();
        thread1.start();
    }
}
```
```java
public class TestTread4 {
    public static void main(String[] args) {
        Company company=new Company(new You());
        company.Haapiy();
    }
}
interface Merry{
    void Haapiy();
}
class You implements Merry{

    @Override
    public void Haapiy() {
        System.out.println("今天结婚很开心");
    }
}
class Company implements Merry{
    You you;
    Company(You o){
        this.you=o;
    }
    @Override
    public void Haapiy() {
        System.out.println("婚庆公司帮你布置场地");
        you.Haapiy();
        System.out.println("付婚庆公尾款");
    }
}
```
## 代理
```java
public class TestTread4 {
    public static void main(String[] args) {
        Company company=new Company(new You());
        company.Haapiy();
    }
}
interface Merry{
    void Haapiy();
}
class You implements Merry{

    @Override
    public void Haapiy() {
        System.out.println("今天结婚很开心");
    }
}
class Company implements Merry{
    You you;
    Company(You o){
        this.you=o;
    }
    @Override
    public void Haapiy() {
        System.out.println("婚庆公司帮你布置场地");
        you.Haapiy();
        System.out.println("付婚庆公尾款");
    }
}
```
## 函数式表达式
```java
public class TestTread5 {
    //3静态内部类
    static class Likee2 implements Like{
        @Override
        public void lambda() {
            System.out.println("i do not like lambda3");
        }
    }
    public static void main(String[] args) {
        Like like=new Likee();
        like.lambda();
        Likee2 likee2=new Likee2();
        likee2.lambda();
        //4.局部内部类
        class Likee3 implements Like{
            @Override
            public void lambda() {
                System.out.println("i do not like lambda4");
            }
        }
        like=new Likee3();
        like.lambda();
        //5.匿名内部类
        like=new Like() {
            @Override
            public void lambda() {
                System.out.println("i do not like lambda5");
            }
        };
        like.lambda();
        //6.用lambda简化
        like=()->{
            System.out.println("i do not like lambda6");
        };
        like.lambda();
    }
}
//1定义一个函数式接口
interface Like{
    void lambda();
}
//2实现类
class Likee implements Like{
    @Override
    public void lambda() {
        System.out.println("i do not like lambda1");
    }
}
```
## 测试stop
```java
//测试stop
//1.建议线程正常停止--》利用次数，不建议死循环
//2.建议使用标志位--设置一个标志位
//3.不要使用stop或者destory等过时jdk不建议得方法
//模拟网络延时:放大问题得发生性
public class TreadTread6 implements Runnable{
    @Override
    public void run() {

    }
    public void take(){

    }

    public static void main(String[] args) throws InterruptedException {
     Date date=new Date(System.currentTimeMillis());
     int i=0;
     while (i<=10){
         ++i;
         Thread.sleep(1000);
         System.out.println(new SimpleDateFormat("MM:hh:ss").format(date));
         date=new Date(System.currentTimeMillis());
     }
    }
}
```
## 测试线程优先级
```java
//测试线程优先级
public class TestTread8{
    public static void main(String[] args) {
        MyPriority myPriority=new MyPriority();
        MyPriority myPriority1=new MyPriority();
        MyPriority myPriority2=new MyPriority();
        MyPriority myPriority3=new MyPriority();
        Thread thread=new Thread(myPriority,"哈哈");
        Thread thread1=new Thread(myPriority,"哈哈1");
        Thread thread2=new Thread(myPriority,"哈哈2");
        Thread thread3=new Thread(myPriority,"哈哈3");
        thread.setPriority(10);
        thread.start();
        thread1.setPriority(1);
        thread1.start();
        thread2.setPriority(5);
        thread2.start();
        thread3.setPriority(1);
        thread3.start();
    }
}
class MyPriority  implements Runnable{

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName()+"-->"+Thread.currentThread().getPriority());
    }
}
```
## 死锁
```java
public class TestTread9 {
    public static void main(String[] args) {
        Makeup makeup=new Makeup(0,"灰姑娘");
        Makeup makeup1=new Makeup(1,"白雪公主");
        Thread thread=new Thread(makeup);
        Thread thread1=new Thread(makeup1);
        thread.start();
        thread1.start();
    }
}
//口红
class Lipstick{

}
//镜子
class Mirror{

}
class Makeup extends Thread{
    //需要的资源只有一份,用static来保证只有一份
    static Lipstick lipstick=new Lipstick();
    static Mirror mirror=new Mirror();

    //选择
    int chose;
    //使用化妆品的人
    String girlName;
    Makeup(int chose,String girlName){
        this.chose=chose;
        this.girlName=girlName;
    }
    @Override
    public void run() {
        //化妆
        try {
            makeup();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    //化妆,要互相持有对方的锁,就是需要拿到对方的资源
    private void makeup() throws InterruptedException {
     if (chose==0){
         synchronized (lipstick){
             System.out.println("获得口红的锁:"+this.girlName);
             Thread.sleep(1000);
         synchronized (mirror){
             System.out.println("获得镜子的锁:"+this.girlName);
         }
         }
     }else{
         synchronized (mirror){
             System.out.println("获得镜子的锁:"+this.girlName);
             Thread.sleep(2000);
             synchronized (lipstick){
                 System.out.println("获得口红的锁:"+this.girlName);
             }
         }
     }
    }
}
```
## 锁
```java
public class TestLock10 implements Runnable{
    int a=10;
    protected final ReentrantLock lock=new ReentrantLock();
    @Override
    public void run() {
        try {//上鎖
            lock.lock();
            while (a>=0){
                System.out.println("lock的使用"+a--);
            }
        }finally {
            //解鎖
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        TestLock10 testLock10=new TestLock10();
        Thread thread=new Thread(testLock10);
        Thread thread1=new Thread(testLock10);
        Thread thread2=new Thread(testLock10);
        thread.start();
        thread1.start();
        thread2.start();
    }
}
```
```java
public class mianshiti{
    public static void main(String[] args) {
        CD cd=new CD();
        for (int i = 1; i <=3; i++) {
            Thread thread=new Thread(cd);
            thread.start();
            cd.fu();
        }
    }
}
class CD implements Runnable{
    boolean flag=false;
    @Override
    public void run() {
        zhi();
    }
    public synchronized void zhi(){
            if (flag){
                try {
                    wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            for (int i = 1; i <= 10; i++) {
                System.out.println("子线程运行次数："+i);
            }
            flag=!flag;
            notify();
    }
    public synchronized void fu(){
            if (!flag){
                try {
                    wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            for (int i = 1; i <= 3; i++) {
                System.out.println("父线程运行次数："+i);
            }

            notify();
    }
}
```
## 礼让线程
```java
//测试礼让线程
//礼让不一定成功,看CPU心情
public class TestTreadYield implements Runnable{
    @Override
    public void run() {
        System.out.println("线程A在运行");
        System.out.println("线程A结束");
    }

    public static void main(String[] args) {
        MyYield myYield=new MyYield();
        Thread threadyeild=new Thread(myYield);
        threadyeild.start();
        TestTreadYield testTreadYield=new TestTreadYield();
        Thread thread=new Thread(testTreadYield);
        thread.start();

    }
}
class MyYield implements Runnable{

    @Override
    public void run() {
        System.out.println("线程正常运行");
        Thread.yield();
        System.out.println("线程结束");
    }
}
```
## 间隔 4 秒执行一次，再间隔 2 秒执行一次，以此类推执行
```java
public class dingshiqi {
    public static void main(String[] args) {
        A a=new A();
        Thread t=new Thread(a);
        t.start();
    }
}
class A implements Runnable{
    Boolean flag=true;
    @Override
    public void run() {
        while (true){
            if(flag){
                four();
                try {
                    Thread.sleep(4000);
                    flag=!flag;
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }else {
                two();
                try {
                    Thread.sleep(2000);
                    flag=!flag;
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    void four(){
        System.out.println("4秒后再执行");
    }
    void two(){
        System.out.println("2秒后再执行");
    }
}
```


