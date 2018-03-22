---
layout: post
title: Java 序列化
date: 2018-03-22 08:00:20 +0300
description: Java 序列化
img: i-rest.jpg # Add image post (optional)
tags: [Java]
---

- [前言](#前言)
- [示例](#示例)

---

#### 前言

Java序列化就是把对象转化为二进制文件。在通过反序列化将二进制文件转化为对象。
持久化对象，在必要的时候反序列化回来。比如在应用停止前保存一些对象，在应用启动的时候在通过反序列化恢复到之前状态。

---

#### 示例

在java中，要实现序列化，只需要在实现接口Serializable
```java
public class SerializableDemo {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        FileOutputStream fos = new FileOutputStream("/opt/fit2cloud/test.txt");
        ObjectOutputStream oos = new ObjectOutputStream(fos);

        Person person = new Person();
        person.setAge(18);
        person.setName("dongbin");
        person.setSex("man");

        oos.writeObject(person);
        oos.flush();
        oos.close();

        FileInputStream fis = new FileInputStream("/opt/fit2cloud/test.txt");
        ObjectInputStream ois = new ObjectInputStream(fis);
        Person object = (Person) ois.readObject();
        System.out.println(object.getAge()+":"+object.getName()+":"+object.getSex());
    }
}

class Person implements Serializable {

    private static final long serialVersionUID = 1L;

    private static int a = 5;

    private String name;
    private String sex;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```


注意事项：
- 反序列化时一定要找到对应的class文件。
- 注意静态变量是不会被序列化，序列化是对对象来说，静态是属于类级别的。
- 在反序列化的时候，会比较serialVersionUID和class是否对应，如果不一致则不能反序列化成功。
- 如果子类序列化,父类没有序列化(要有默认空构造方法)。父类参数会序列化道子类，但是没有值。
- transient关键字 可以阻止变量序列化到文件。