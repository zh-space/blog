---
title: 字节流和字符流的学习 study
date: 2022-2-11 
tags:
 - java
 - 字节流和字符流
---
## 以前只是知道字节流字符流。今天深入学习了一下字节流字符流
参考博客 https://blog.csdn.net/qq_35122713/article/details/88793019
```java
/**
     * 字节流和字符流的区别
     * @throws Exception
     */
    @Test
    public void test25() throws Exception{
        // 第1步：使用File类找到一个文件
        File f=new File("C:\\Users\\Administrator\\Desktop\\测试输入输出流.txt");
        // 第2步：通过子类实例化父类对象  // 准备好一个输出的对象
        OutputStream outputStream=new FileOutputStream(f);
        // 通过对象多态性进行实例化
        // 第3步：进行写操作
        String s="输出流-字节流";
        // 准备一个字符串
        byte b[]=s.getBytes();
        // 字符串转byte数组
        outputStream.write(b);
        //关闭输出流
        outputStream.close();
    }
    @Test
    public void test26() throws Exception{
        // 第1步：使用File类找到一个文件
        File f=new File("C:\\Users\\Administrator\\Desktop\\测试输入输出流.txt");
        String s="输出流-字符流";
        // 第2步：通过子类实例化父类对象  // 准备好一个输出的对象
        /*第一种*/
        OutputStream outputStream=new FileOutputStream(f);
        OutputStreamWriter outputStreamWriter=new OutputStreamWriter(outputStream);//将字节流转换为字符流
        outputStreamWriter.write(s);
        /*第二种*/
        FileWriter fileWriter=new FileWriter(f);
        fileWriter.write(s);
        //关闭输出流
        outputStreamWriter.close();
        //outputStreamWriter.flush();
        fileWriter.close();
        //fileWriter.flush();
        /*
        程序运行后会发现文件中没有任何内容，这是因为字符流操作时使用了缓冲区，而 在关闭字符流时会强制性地将缓冲区中的内容进行输出，
        但是如果程序没有关闭，则缓冲区中的内容是无法输出的，所以得出结论：字符流使用了缓冲区，而字节流没有使用缓冲区。
        提问：什么叫缓冲区？
        在很多地方都碰到缓冲区这个名词，那么到底什么是缓冲区？又有什么作用呢？
        回答：缓冲区可以简单地理解为一段内存区域。
        可以简单地把缓冲区理解为一段特殊的内存。
        某些情况下，如果一个程序频繁地操作一个资源（如文件或数据库），则性能会很低，此时为了提升性能，就可以将一部分数据暂时读入到内存的一块区域之中，
        以后直接从此区域中读取数据即可，因为读取内存速度会比较快，这样可以提升程序的性能。
        在字符流的操作中，所有的字符都是在内存中形成的，在输出前会将所有的内容暂时保存在内存之中，所以使用了缓冲区暂存数据。
        如果想在不关闭时也可以将字符流的内容全部输出，则可以使用Writer类中的flush()方法完成。
        此时，文件中已经存在了内容，更进一步证明内容是保存在缓冲区的。这一点在读者日后的开发中要特别引起注意。
提问：使用字节流好还是字符流好？
学习完字节流和字符流的基本操作后，已经大概地明白了操作流程的各个区别，那么在开发中是使用字节流好还是字符流好呢？
回答：使用字节流更好。
在回答之前，先为读者讲解这样的一个概念，所有的文件在硬盘或在传输时都是以字节的方式进行的，包括图片等都是按字节的方式存储的，而字符是只有在内存中才会形成，
所以在开发中，字节流使用较为广泛。
        * */
    }
    @Test
    public void test27() throws Exception{
        File f=new File("C:\\Users\\Administrator\\Desktop\\测试输入输出流.txt");
        InputStream inputStream=new FileInputStream(f);
        byte[] b = new byte[2];
        int i=0;
        while ((i=inputStream.read(b))!=-1){
            String read = new String(b);
            System.out.println(read);
        }
    }
    @Test
    public void test28() throws Exception{
        File f=new File("C:\\Users\\Administrator\\Desktop\\测试输入输出流.txt");
        InputStream inputStream=new FileInputStream(f);
        InputStreamReader inputStreamReader=new InputStreamReader(inputStream);
        byte[] b = new byte[2];
        int i=0;
        while ((i=inputStreamReader.read())!=-1){
            String read = new String(b);
            System.out.println(read);
        }
    }
    /*将一个java对象序列化到文件里面*/ 在 java 中能够被序列化的类必须先实现 Serializable 接口，该接口没有任何抽象方法只是起到一个标记作用。
    @Test
    public void test29() throws Exception{
        File f=new File("F://obj");
        OutputStream outputStream=new FileOutputStream(f);
        ObjectOutputStream objectOutputStream=new ObjectOutputStream(outputStream);
        User user=new User();
        user.setAge(1);
        user.setName("z");
        objectOutputStream.writeObject(user);
        objectOutputStream.close();
    }
    @Test
    public void test30() throws Exception{
        File f=new File("F://obj");
        InputStream inputStream=new FileInputStream(f);
        ObjectInputStream objectInputStream=new ObjectInputStream(inputStream);
        User user=(User) objectInputStream.readObject();
        System.out.println(user);
    }
```
字节流读取的时候，读到一个字节就返回一个字节； 字符流使用了字节流读到一个或多个字节（中文对应的字节
数是两个，在 UTF-8 码表中是 3 个字节）时。先去查指定的编码表，将查到的字符返回。 字节流可以处理所有类型数
据，如：图片，MP3，AVI 视频文件，而字符流只能处理字符数据。只要是处理纯文本数据，就要优先考虑使用字符
流，除此之外都用字节流。字节流主要是操作 byte 类型数据，以 byte 数组为准，主要操作类就是 OutputStream、
InputStream
字符流处理的单元为 2 个字节的 Unicode 字符，分别操作字符、字符数组或字符串，而字节流处理单元为 1 个字
节，操作字节和字节数组。所以字符流是由 Java 虚拟机将字节转化为 2 个字节的 Unicode 字符为单位的字符而成的，
所以它对多国语言支持性比较好！如果是音频文件、图片、歌曲，就用字节流好点，如果是关系到中文（文本）的，用
字符流好点。在程序中一个字符等于两个字节，java 提供了 Reader、Writer 两个专门操作字符流的类。