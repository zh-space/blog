---
title: javaweb自学笔记
date: 2021-07-16 
tags:
 - javaweb
---
## 自学前篇
因为自己实训的时候选的微信小程序，没有选择javaweb，有点后悔，因为感觉微信小程序比较简单，而javaweb能学到更多东西。所以自己暑假准备自学javaweb并且做个项目出来。首先当然是看b站的教学视频啦，有老师教还是比自己自学快的。这篇文章就和大家分享我的自学过程，还有笔记。
## javascript的表单
form标签是表单标签
  action属性设置提交的服务器地址
  method属性设置提交的方式get/post
表单提交的时候，数据没有发送给服务器的三种情况
  1.表单没有name属性值
  2.单选复选（下拉列表的option标签都需要添加value属性，以便发送给服务器）
  3.表单项不在提交的form标签中
POST的特点：
  提交数据大小理论上没有要求
  数据类型没有要求
  安全性更高
  响应速度稍慢
GET的特点：
  响应速度快
  只能是字符串
  数据在请求头中安全性较低
  GET能传递的数据字节短
```html
<form action="http://locationhost:8080" method="GET">
        <input type="hidden" name="action" value="login"/>
        <table>
            <tr>
                <td>用户名称：</td>
                <td>
                    <input type="text" name="username" value="默认值"/></td>
            </tr>
            <tr>
                <td>用户密码：</td>
                <td><input type="text" name="password" value=""/></td>
            </tr>
        </table>
    </form>
```
## javascript的运算符和java有些不同
&&且运算
  有两种情况：
  第一种：当表达式全为真的时候，返回最后一个表达式的值
  第二种：当表达式中有一个为假的时候，返回第一个为假的表达式
```js
    var a="abc";
    var b=true;
    var c=false;
    var d=null;
    //alert(a && b);  true
    //alert(b && a);  true
    //alert(a && d);  false
    //alert(a && c);  null
    //alert(a && d && c);  false
```
||且运算
  有两种情况：
  第一种：当表达式全为假的时候，返回最后一个表达式的值
  第二种：当表达式中有一个为真的时候，返回第一个为真的表达式
```js
    var a="abc";
    var b=true;
    var c=false;
    var d=null;
    //alert(d || c);  null
    //alert(c || d);  false
    //alert(a || c);  abc
    //alert(b || c);  true
```
## javascript的数组和python的数组很相似，略过。
略
## javascrpit定义函数的两种方式
```js
function giao(){
        alert("giao")
    }
    var giaogiao=function(){
        alert("giaogiao")
    }
    giao();
    giaogiao();
```
另外因为js是弱类语言，所以要定义有参函数在小括号里面(a,b)就可以了和python一样.
## js不支持重载
## 函数的arguments隐形参数（仅在function内）
```js
function fun(a){
        var result=0;
        alert(arguments.length);
        alert(arguments[0]);
        alert(arguments[1]);
        alert(arguments[2]);
        alert("a="+a)
        for (var i=0;i<arguments.length;i++){
            result+=arguments[i];
        }
        return result
    }
fun(1,2,3,4,5,6,7,8,9)
```
## javascript的自定义对象
对象的定义:
  var 变量名 = new Object(); //对象实例（空对象）
  变量名.属性名 = 值；        //定义一个属性
  变量名.函数 = function(){} //定义一个函数
对象的访问:
  变量名.属性/函数名();

对象的定义:
  var 变量名 = {
      属性名: 值，
      属性名: 值，
      函数名: function(){}
  }；
对象的访问:
  变量名.属性/函数名();
## javascript中的事件
常用的事件:
  onload 加载完成事件        页面加载完成之后，常用于做页面js代码初始化操作
  onclick 单击事件          常用于按钮的点击响应操作
  onblur 失去焦点事件        常用于输入框失去焦点后验证其输入内容是否合法
  onchange 内容发送改变事件  常用于下拉列表和输入框内容发生改变后操作
  onsubmit 表单提交事件      常用于表单提交前，验证所有表单项是否合法
## javascript静态注册与动态注册
onload事件
```js
<script type="text/javascript">
  // onLoad事件的方法
  function onloadFun() {
      alert('静态注册onload事件，所有代码");
      }
      // onLoad事件动态注册。是固定写法
      window.onload = function (){
          alert("动态注册的onload事件");
      }
</script>
<!--静态注册onload事件
onLoad事件是浏览器解析完页面之后就会自动触发的事件
<body onload="onLoadFun( ); ">
-->
```
onclick事件
```html
<html lang="en">
<head>
<meta charset="UTF-8"><title>Title</title>
<script type="text/javascript">
function onclickFun() {
    alert("静态注册onclick事件");
    }
    //动态注册onclick事件
    window.onload = function ()
    //1获取标签对象
    /*
    * document是JavaScript语言提供的一个对象（文档)<br/>
    * get    获取
    * ELement 元亲（就是标签）
    *By   通过。。由。。经。e :
    * Id   id层性
    getELementById通过id层性获取标签对翻**/
    var btn = document.getElementById("giao");
    btn.onclick = funtion(){
        alert("动态注册onclick事件")
    }
    //2通过标签对象.事件名= function(){}
    }
    </script>
</head>
<body>
    <!--静态注册onclick事件-->
    <button onclick="onclickkFun()">静态注册</button>
    <!--动态注册onclick事件-->
    <button id="giao">动态注册</button>
</body>
</html>
```
onblur onchange onsubmit事件类似这里不多讲.
## 正则表达式
表示要求字符串中，是否包含字母e
var patt = new RegExp("e");
var patt = /e/; 也是正则表达式对象
表示要求字符串中，是否包含字母a或b或c
var patt = / [abc] / ;
表示要求字符串，是否包含小写字母
var patt = / [a-z]/ ;
表示要求字符串，是否包含任意大写字母
var patt = /[A-Z]/ ;
表示要求字符串，是否包含任意数字
var patt = / [0-9] / ;
表示要求字符串，是否包含字母，数字，下划线
var patt = /\w/ ;
表示要求字符串中是否包含至少―个a
var patt = /a+/ ;
表示要求字符串中是否包含零个或多个a
var patt = /a*/ ;
表示要求字符串是否包含一个或零个a
var patt = /a? / ;
表示要求字符串是否包含连续三个a
ar patt = /a{3}/ ;
表示要求字符串是否包至少3个连续的a，最多5个连续的a
var patt = /a{3,5}/ ;
表示要求字符串是否包至少3个连续的a,
var patt = /a{3,}/;
表示要求字符串必须以a结尾
var patt= /a$/
表示要求字符串必须以a开头
var patt= /^a/
## jQuery的学习
首先导入jQuery包，可到官网下载
```js
<script src="jquery-3.6.0.min.js"></script>
```
```html
<body>
    <button id="btn">BUTTON</button>
</body>
<script>
    // window.onload=function(){
    //     var a = document.getElementById("btn");
    //     a.onclick=function(){
    //         alert("giao")
    //     }
    // }
    //表示页面加载之后，相当于window.onload=function(){}
    $(function(){
        var a=$("button");//表示按照id查询标签对象，相当于document.getElementById("btn");$(标签名)，按照标签查询.$(.class)按照class属性查询.
        a.click(function(){ //绑定单机事件
            alert("giao")
        });
    });
</script>
```
另外要注意的是,jQuery对象的方法和DOM对象的方法是不适用的,比如说jQuery用click  DOM用onclick
## DOM对象与jQuery对象的相互转换
1.先有DOM对象
2.$(DOM对象); var dom=$(DOM对象)  就可以转换为jQuery对象了

1.先有jQuery对象
2.jQuery下标可取 var DOM=dom[下标]
## jQuery组合选择器
$("div,span,giao.myclass,giao#myclass")
## jQuery层级选择器
$("form span")
$("form>span") 匹配子元素
$("label+input") 匹配跟在label后面的input元素
$("form~input") 匹配form后的所有input元素
$("li:first") 匹配li标签的第一个元素
$("li:last") 匹配li标签的最后一个元素
$("input:not(:checked)") 匹配没有选中的input元素
$("tr:even") 匹配索引为偶数的元素
$("tr:odd") 匹配索引为奇数的元素
$("tr:odd") 匹配索引为奇数的元素
$("tr:eq(1)") 匹配tr的第二行
$("tr:gt(3)") 匹配索引大于3的元素
$("tr:lt(3)") 匹配索引小于3的元素
$(":header") 匹配标题元素
$(":animated") 匹配正在执行动画的元素
## 文本过滤器
$("div:contains("giao")"·) 匹配内容里面包含giao的div元素
$("td:empty") 匹配内容为空的
$("td:parent") 匹配内容不为空的
$("div:has(p).addClass("test")") 给所以包含p标签的div添加test类属性
$("div[id]")  匹配含有id属性的div元素
$("input[name='giao']").attr("checked",true)  匹配含有name=giao属性并且checked为true的input元素
$("input[name!='giao']").attr("checked",true)  匹配不含有name=giao属性并且checked为true的input元素
## 表单过滤器
$(":input") 匹配表单元素 冒号后面可以是text password radio等等type值
还有一些懒得写了,要用再查吧,我giao.
## each
```js
$(function(){
        var aaa=$("button");
        for(i=0;i<aaa.length;i++){
            //alert(aaa[i].innerHTML);
            alert(aaa[i].value);
        }
    })
```
这里要用innerhtml才行
```js
$(function(){
        var aaa=$("button");
        aaa.each(function(){
            alert(this.value);
        })
    })
```
使用jQuey里面的each函数使用this.valu就可以获得值了
## 元素的筛选 太难写了,贴图好了.
![自己看](https://i.loli.net/2021/07/21/oy6YwjRxncLIWEt.png)
## jQuery属性操作
批量操作单选
```js
$(":radio").val(["radio1","radio2"]);
$(":checkbox").val(["checkbox1","checkbox2"]);
```
$(":checkbox:first").attr("name","giao");可以修改name的值为giao
attr() 不推荐操作checked,readOnl,selected,disabled等
attr() 还可以操作自定义属性
prop() 推荐操作checked,readOnl,selected,disabled等
## 插入操作
元素插入
```js
$(function(){
        $("<p>hello</p>").appendTo("div");
    })
```
元素替换
```js
a.replaceWith(b);  用b替换a
a.replaceAll(b);  用a替换所有b
```
```js
$(function(){
        $("div").replaceWith($("<p>hello</p>"));
    })
```
元素删除
```js
a.remove(); 删除a标签
a.empty();  清空a标签里的内容
```
## jQuery动画
![giao](https://i.loli.net/2021/07/21/s3LpYIQ7doj6T1X.png)
## jQuery事件操作
1.jQuery的页面加载完成后是浏览器内核解析完页面标签创建好的DOM对象后马上执行
2.元素js的页面加载完成后，除了等浏览器内核解析完标签创建好DOM对象，还要等标签显示内容加载完成
jQuery页面加载完成后再执行源生window.load
1.原生js页面加载完成之后。只会执行最后一次赋值函数
2.jQuery页面加载完成之后，会把注册的function函数所有的函数都依次执行
## 小实验，放大图片
![giao](https://i.loli.net/2021/07/21/oAF8sPhik3z6wtv.png)
## xml如何插入特殊字符
CDATA语法  <![CDATA[里面放文本内容]]]>
## 如何用dom4j解析xml文件
```java
package com.atuigu.pojo;

import org.dom4j.Document;
import org.dom4j.DocumentException;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;
import org.junit.Test;

import java.util.List;

public class Dom4jTest {
    @Test
    public void test1() throws Exception {
        //创建一个SAXReader输入流，去读取xml配置文件，生成Document对象。
        SAXReader saxReader=new SAXReader();
        try {
            Document document=saxReader.read("xml/src/books.xml");
            System.out.println(document);
        }catch (Exception e){
            e.printStackTrace();
        }
    }
    @Test
    public void test2() throws DocumentException {
        //1 读取books.xml文件
        SAXReader reader=new SAXReader();
        //在junit测试中相对路径是从模块名开始
        Document document = reader.read("xml/src/books.xml");
        //2 通过Document对象获取根元素 Element是用来构建中节点的
        Element rootElement=document.getRootElement();
        //System.out.println(rootElement);
        //3 通过根元素获取book标签对象 element elements都是通过标签名查找元素
        List<Element> books=rootElement.elements("book");
        //4 遍历，处理每个标签转换为Book类
        for(Element book : books){
            //asXML()把标签转换为book类
            //Element nameElement=book.element("name");
            //String nameText=nameElement.getText();
            //System.out.println(nameElement.asXML());
            //getText()：
            //直接获取指定标签名的内容
            String nameText=book.elementText("name");
            String author=book.elementText("author");
            String price=book.elementText("price");
            //获取属性值
            String snvalue=book.attributeValue("sn");
            //System.out.println(nameText+author+price+snvalue);
            System.out.println(new Book(snvalue,nameText,Double.parseDouble(price),author));
        }
    }
}

```
Book类
```java
package com.atuigu.pojo;
import java.math.BigDecimal;
public class Book {
    private String sn;
    private String name;
    private BigDecimal price;
    private String author;

    public String getSn() {
        return sn;
    }

    public void setSn(String sn) {
        this.sn = sn;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public BigDecimal getPrice() {
        return price;
    }

    public void setPrice(BigDecimal price) {
        this.price = price;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    public Book(String sn, String name, Double price, String author) {
        this.sn = sn;
        this.name = name;
        this.price = BigDecimal.valueOf(price);
        this.author = author;
    }

    @Override
    public String toString() {
        return "Book{" +
                "sn='" + sn + '\'' +
                ", name='" + name + '\'' +
                ", price=" + price +
                ", author='" + author + '\'' +
                '}';
    }
}

```
解析的xml文件
```xml
<?xml version="1.0" encoding="utf-8"?>
<books>
 <book sn="SN123">
  <name>时间简史</name>
  <author>霍金</author>
  <price>75</price>
 </book>
 <book sn="SN1234">
  <name>java从入门到放弃</name>
  <author><![CDATA[<gaio桑>]]></author>
  <price>250</price>
 </book>
</books>
```
## 如何在idea上创建动态web工程
首先点击file ->设置 ->构建,执行,部署->应用程序服务器 然后点击+号，添加本地tomcat并选择路径
![giao](https://i.loli.net/2021/07/22/ZHDXm1tVNdF62PL.png)
如果你是新版的idea需要先建一个java项目再右键添加框架支持勾选applicationweb.再点击完成,就完成啦!
## Servlet的学习
javaweb三大组件Servlet程序，Filter过滤器，Listener监听器。
新建一个java文件接口选择implement servlet
然后在配置文件里面配置servlet
```xml
 <!--servlet标签给tomcat配置servlet程序-->
    <servlet>
        <!--servlet-name标签servlet程序起一个别名(HelloServlet)-->
        <servlet-name>HelloServlet</servlet-name>
        <!--servlet-class标签是servlet程序的全类名-->
        <servlet-class>com.atguigu.servlet.HelloServlet</servlet-class>
    </servlet>

    <!--servlet-mapping标签给servlet程序配置访问地址-->
    <servlet-mapping>
        <!--servlet-name标签的作用是告诉服务器给那个servlet使用-->
        <servlet-name>HelloServlet</servlet-name>
        <!--配置访问路径-->
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>
```
在浏览器输入http://localhost:你的端口地址/你的工程路径/hello  通过ip地址定位服务器，通过端口号访问tomcat，再通过工程路径访问工程，再通过/hello到配置文件里面找路径。
比如我的是http://localhost:8888/demo/hello
/hello就是配置的访问路径，输入这个路径就会访问com.atguigu.servlet.HelloServlet下的HelloServlet
## 这个暑假时间有点紧，就不写日志了，有想要更详细的教程的可以联系我


