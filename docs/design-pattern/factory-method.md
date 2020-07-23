### Factory Method模式

#### 定义

定义一个用于创建对象的接口，让子类决定实例化哪一个类。工厂方法是一个类的实例化延迟到其子类。

#### 类图

![工厂方法模式类图](https://www.ultroncode.com/source/86a84731b0a88baa492a425e37811892.png)

#### 示例程序

这段示例程序的作用是制作身份证（ID卡），它其中有5个类。

Product类和Factory类，这两个类组成了生成实例的框架。
IDCard类和IDCardFactory类负责实际的加工处理。
Main类用于测试程序行为的类。

Product类

```java
public abstract class Product {
    public abstract void use();
}
```

Factory类

```java
public abstract class Factory {
    public final Product create(String owner) {
        Product p = createProduct(owner);
        registerProduct(p);
        return p;
    }

    protected abstract Product createProduct(String owner);

    protected abstract void registerProduct(Product product);
}
```

IDCard类

```java
public class IDCard extends Product {
    private String owner;
    IDCard(String owner) {
        System.out.println("制作" + owner + "的ID卡。");
        this.owner = owner;
    }

    @Override
    public void use() {
        System.out.println("使用" + owner + "的ID卡。");
    }

    public String getOwner() {
        return owner;
    }
}
```

IDCardFactory类

```java
public class IDCardFactory extends Factory {
    private List<String> owners = new ArrayList<>();

    @Override
    protected Product createProduct(String owner) {
        return new IDCard(owner);
    }

    @Override
    protected void registerProduct(Product product) {
        owners.add(((IDCard) product).getOwner());
    }

    public List<String> getOwners() {
        return owners;
    }
}
```

Main类

```java
public class Main {
    public static void main(String[] args) {
        Factory factory = new IDCardFactory();
        Product card1 = factory.create("小明");
        Product card2 = factory.create("小红");
        Product card3 = factory.create("小刚");
        card1.use();
        card2.use();
        card3.use();
    }
}
```

输出结果

```console
制作小明的ID卡。
制作小红的ID卡。
制作小刚的ID卡。
使用小明的ID卡。
使用小红的ID卡。
使用小刚的ID卡。
```

#### 相关设计模式

- Templete Method模式
  Factory Method模式是Templete Method模式的典型应用。在示例程序中，create方法就是模版方法。
- Singleton模式
  在多数情况下我们都可以将Singleton模式用于扮演Creator角色（或是ConcreteCreator角色）的类。这是因为在程序中没有必要存在多个Creator角色（或是ConcreteCreator角色）的实例。
- Composite模式
  有时可以将Composite模式用于Product角色（或是ConcreteProduct角色）。
- Iterator模式
  有时，在Iterator模式中使用iterator方法生成Iterator实例时会使用Factory Method模式。

#### 哪里用到了Factory Method模式

- [java.util.Calendar](http://docs.oracle.com/javase/8/docs/api/java/util/Calendar.html#getInstance--)
- [java.util.ResourceBundle](http://docs.oracle.com/javase/8/docs/api/java/util/ResourceBundle.html#getBundle-java.lang.String-)
- [java.text.NumberFormat](http://docs.oracle.com/javase/8/docs/api/java/text/NumberFormat.html#getInstance--)
- [java.nio.charset.Charset](http://docs.oracle.com/javase/8/docs/api/java/nio/charset/Charset.html#forName-java.lang.String-)
- [java.net.URLStreamHandlerFactory](http://docs.oracle.com/javase/8/docs/api/java/net/URLStreamHandlerFactory.html#createURLStreamHandler-java.lang.String-)
- [java.util.EnumSet](https://docs.oracle.com/javase/8/docs/api/java/util/EnumSet.html#of-E-)
- [javax.xml.bind.JAXBContext](https://docs.oracle.com/javase/8/docs/api/javax/xml/bind/JAXBContext.html#createMarshaller--)

#### 参考资料

- 图解设计模式
- 设计模式之禅