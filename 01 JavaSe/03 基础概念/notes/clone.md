## 前言

在阿里Java开发手册中，有这么一条建议：慎用 Object 的 clone 方法来拷贝对象。对象 clone 方法默认是浅拷贝，若想实现深拷贝需覆写 clone 方法实现域对象的深度遍历式拷贝 。Java中的对象拷贝，有浅拷贝和深拷贝两种，如果没有搞清楚这两者的区别，那么可能会给自己的代码埋下隐患。

## 什么是浅拷贝和深拷贝

浅拷贝：**被复制对象的所有变量都含有与原来的对象相同的值，而所有的对其他对象的引用仍然指向原来的对象。**换言之，浅复制仅仅复制所考虑的对象，而不复制它所引用的对象。

深拷贝：**被复制对象的所有变量都含有与原来的对象相同的值，除去那些引用其他对象的变量。**那些引用其他对象的变量将指向被复制过的新对象，而不再是原有的那些被引用的对象。换言之，深复制把要复制的对象所引用的对象都复制了一遍。

通过上面的结论，我们可以看出浅拷贝和深拷贝的区别就在于所要拷贝的对象的引用数据类型，如果是拷贝一份引用，那么这是浅拷贝，如果是新建一个对象，那么这就是深拷贝。

## clone方法

在Java的Object对象中，有clone这个方法。它被声明为了 `protected`，所以我们可以在其子类中使用它。这里需要注意的是，我们在子类中使用clone方法时，子类需要实现Cloneable接口，否则会抛出java.lang.CloneNotSupportedException异常。

## 有如下对象

如下实体类都使用了Lombok。

Address.java

```java
@Data
public class Address {

    private String address;

}
```

Person.java

```java
@Data
public class Person implements Cloneable {

    private String name;

    private Integer age;
    
    private Address address;

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

## 浅拷贝

浅拷贝，示例代码如下：

```java
public static void main(String[] args) throws CloneNotSupportedException {
    Person person = new Person();
    person.setName("Happyjava");
    person.setAge(33);
    Address address = new Address();
    address.setAddress("浙江杭州");
    person.setAddress(address);
    Person newPerson = (Person) person.clone();
    System.out.println(person == newPerson);
    System.out.println(person.getAddress() == newPerson.getAddress());
}
```

通过 == 比较是否是同一个对象。其运行结果如下：

```java
false
true
```

说明了通过clone方法拷贝出来的对象，与原对象并不是同一个对象。而person.getAddress() == newPerson.getAddress() 的比较是true，**说明了二者的引用都是指向同一个对象**。这就是浅拷贝，引用类型还是指向原来的对象。

## 浅拷贝存在的问题

很多时候，我们拷贝一个对象，是希望完全进行深度拷贝的。浅拷贝存在的问题就是，对于原对象引用类型的属性进行修改，拷贝出来的对象也会受到影响（因为二者的引用都指向同一个对象）。如下代码：

```java
public static void main(String[] args) throws CloneNotSupportedException {
    Person person = new Person();
    person.setName("Happyjava");
    person.setAge(33);
    Address address = new Address();
    address.setAddress("浙江杭州");
    person.setAddress(address);
    Person newPerson = (Person) person.clone();
    newPerson.getAddress().setAddress("广东省深圳市");
    System.out.println(person.getAddress().getAddress());
}
```

运行结果如下：

![img](https://user-gold-cdn.xitu.io/2019/8/1/16c4b1151a057837?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

通过newPerson把address设置为“广东省深圳市”，person的address也变成了"广东省深圳市"。

这种情况，如果我们没有注意，是很容易造成生产事故的。

## 深拷贝

通过clone方法实现深拷贝，是一件比较麻烦的事情，因为我们需要手动在clone方法里拷贝引用类型。代码修改如下：

Address.java

```java
@Data
public class Address implements Cloneable {

    private String address;

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

Person.java

```java
@Data
public class Person implements Cloneable {

    private String name;

    private Integer age;

    private Address address;

    @Override
    protected Object clone() throws CloneNotSupportedException {
        Person newPerson = (Person) super.clone();
        newPerson.address = (Address) this.address.clone();
        return newPerson;
    }
}
```

通过clone方法实现深拷贝，我们需要在Person的clone方法里调用address的clone方法，并且手动设置clone出来的新的address。

再次执行上面的测试代码，运行结果如下：

![img](https://user-gold-cdn.xitu.io/2019/8/1/16c4b11519fd52a4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 通过序列化实现深拷贝

通过clone方法实现深拷贝是比较麻烦的一件事情，这里推荐大家可以通过序列化、反序列化的方式实现深拷贝。我们可以直接使用commons-lang3包的序列化、反序列工具类。

引入依赖

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.8.1</version>
</dependency>
```

序列化需要实现Serializable接口，Person和Address类都需要实现。测试代码如下：

```java
public static void main(String[] args) throws CloneNotSupportedException {
    Person person = new Person();
    person.setName("Happyjava");
    person.setAge(33);
    Address address = new Address();
    address.setAddress("浙江杭州");
    person.setAddress(address);
    // 序列化
    byte[] serialize = SerializationUtils.serialize(person);
    // 反序列化
    Person newPerson = SerializationUtils.deserialize(serialize);
    System.out.println(person == newPerson);
    System.out.println(person.getAddress() == newPerson.getAddress());
}
```

运行结果如下：

![img](https://user-gold-cdn.xitu.io/2019/8/1/16c4b1151a12fcbd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

通过结果可以看出，反序列化构建出来的对象，是全新的、深度拷贝的。

## 总结

拷贝对象，如果直接通过clone方法进行拷贝，是很容易出现问题的。我们要清楚的知道浅拷贝和深拷贝的区别。

作者：happyjava链接：https://juejin.im/post/5d425230f265da039519d248