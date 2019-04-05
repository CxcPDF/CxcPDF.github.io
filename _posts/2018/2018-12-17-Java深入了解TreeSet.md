---
layout: post
title: Java深入了解TreeSet
categories: [Java]
description: Java深入了解TreeSet
keywords: Java,TreeSet

---

Java中的TreeSet是Set的一个子类，TreeSet集合是用来对象元素进行排序的,同样他也可以保证元素的唯一。
那TreeSet为什么能保证元素唯一，它是怎样排序的呢？先看一段代码：


```java
public static void demo() {
        TreeSet<Person> ts = new TreeSet<>();
        ts.add(new Person("张三", 23));
        ts.add(new Person("李四", 13));
        ts.add(new Person("周七", 13));
        ts.add(new Person("王五", 43));
        ts.add(new Person("赵六", 33));
        
        System.out.println(ts);
    }
```

执行结果：

>```
>出错，会抛出一个异常：java.lang.ClassCastException
>显然是出现了类型转换异常。原因在于我们需要告诉TreeSet如何来进行比较元素，如果不指定，就会抛出这个异常
>```

如何解决：
如何指定比较的规则，需要在自定义类(Person)中实现```Comparable```接口，并重写接口中的compareTo方法

```java
public class Person implements Comparable<Person> {
    private String name;
    private int age;
    ...
    public int compareTo(Person o) {
        return 0;                //当compareTo方法返回0的时候集合中只有一个元素
        return 1;                //当compareTo方法返回正数的时候集合会怎么存就怎么取
        return -1;                //当compareTo方法返回负数的时候集合会倒序存储
    }
}
```

  为什么返回0，只会存一个元素，返回-1会倒序存储，返回1会怎么存就怎么取呢？原因在于TreeSet底层其实是一个二叉树机构，且每插入一个新元素(第一个除外)都会调用```compareTo()```方法去和上一个插入的元素作比较，并按二叉树的结构进行排列。
\1. 如果将```compareTo()```返回值写死为0，元素值每次比较，都认为是相同的元素，这时就不再向TreeSet中插入除第一个外的新元素。所以TreeSet中就只存在插入的第一个元素。
\2. 如果将```compareTo()```返回值写死为1，元素值每次比较，都认为新插入的元素比上一个元素大，于是二叉树存储时，会存在根的右侧，读取时就是正序排列的。
\3. 如果将```compareTo()```返回值写死为-1，元素值每次比较，都认为新插入的元素比上一个元素小，于是二叉树存储时，会存在根的左侧，读取时就是倒序序排列的。

利用上述规则，我们就可以按照年龄来排序了。代码如图  

```java
public int compareTo(Person o) {
        int num = this.age - o.age;                //年龄是比较的主要条件
        return num == 0 ? this.name.compareTo(o.name) : num;//姓名是比较的次要条件
    }
```

按照姓名排序(依据Unicode编码大小),代码如下：

```java
public int compareTo(Person o) {
        int num = this.name.compareTo(o.name);        //姓名是主要条件
        return num == 0 ? this.age - o.age : num;    //年龄是次要条件
    }
```

按照姓名长度排序,代码如下：

```java
public int compareTo(Person o) {
        int length = this.name.length() - o.name.length();                //比较长度为主要条件
        int num = length == 0 ? this.name.compareTo(o.name) : length;    //比较内容为次要条件
        return num == 0 ? this.age - o.age : num;                        //比较年龄为次要条件
    }
```

  以上是TreeSet如何比较自定义对象，接下来我们再来看一下TreeSet如何利用比较器比较元素。

需求:现在要制定TreeSet中按照String长度比较String。  

```java
//定义一个类，实现Comparator接口，并重写compare()方法，
class CompareByLen /*extends Object*/ implements Comparator<String> {

    @Override
    public int compare(String s1, String s2) {        //按照字符串的长度比较
        int num = s1.length() - s2.length();        //长度为主要条件
        return num == 0 ? s1.compareTo(s2) : num;    //内容为次要条件
    }
}
```

```java
  public static void demo4() {

        //需求:将字符串按照长度排序
        TreeSet<String> ts = new TreeSet<>(new CompareByLen());        //Comparator c = new CompareByLen();
        ts.add("aaaaaaaa");
        ts.add("z");
        ts.add("wc");
        ts.add("nba");
        ts.add("cba");
        
        System.out.println(ts);
    }
```

#### 总结

1. 特点
   - TreeSet是用来排序的, 可以指定一个顺序, 对象存入之后会按照指定的顺序排列
2. 使用方式
   - a.自然顺序(Comparable)
     - TreeSet类的add()方法中会把存入的对象提升为Comparable类型
     - 调用对象的compareTo()方法和集合中的对象比较
     - 根据compareTo()方法返回的结果进行存储
   - b.比较器顺序(Comparator)
     - 创建TreeSet的时候可以制定 一个Comparator
     - 如果传入了Comparator的子类对象, 那么TreeSet就会按照比较器中的顺序排序
     - add()方法内部会自动调用Comparator接口中compare()方法排序
     - 调用的对象是compare方法的第一个参数,集合中的对象是compare方法的第二个参数
   - c.两种方式的区别
     - TreeSet构造函数什么都不传, 默认按照类中Comparable的顺序(没有就报错ClassCastException)
     - TreeSet如果传入Comparator, 就优先按照Comparator