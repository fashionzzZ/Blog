### Template Method模式

#### 定义

在父类中定义处理流程的框架，在子类中实现具体处理的模式称为Templete Method模式。

#### 类图

![模版方法模式类图](https://www.ultroncode.com/source/7adc63490ea64209e786f3fac269b281.png)

#### 示例程序

在示例程序中出现AbstractDisplay、CharDisplay、StringDisplay、Main这4个类。
在`AbstractDisplay`类中定义了`display`方法，而且该方法中一次调用了`open`、`print`、`close`这3个方法。虽然这3个方法已经在AbstractDisplay中声明了，但都是没有实体的抽象方法。
这里，调用抽象方法的`display`方法就是`模版方法`。

#### 示例程序类图

![示例程序类图](https://www.ultroncode.com/source/8d9333517497c7572a8a5e774e1da247.png)

AbstractDisplay.java

```java
public abstract class AbstractDisplay {
    public abstract void open();
    public abstract void print();
    public abstract void close();

    public final void display() {
        open();
        for (int i = 0; i < 5; i++) {
            print();;
        }
        close();
    }
}
```

CharDisplay.java

```java
public class CharDisplay extends AbstractDisplay {
    private char ch;

    public CharDisplay(char ch) {
        this.ch = ch;
    }

    @Override
    public void open() {
        System.out.print("<<");
    }

    @Override
    public void print() {
        System.out.print(ch);
    }

    @Override
    public void close() {
        System.out.println(">>");
    }
}
```

StringDisplay.java

```java
public class StringDisplay extends AbstractDisplay {
    private String string;
    private int width;

    public StringDisplay(String string) {
        this.string = string;
        this.width = string.getBytes().length;
    }

    @Override
    public void open() {
        pringLine();
    }

    @Override
    public void print() {
        System.out.println("|" + string + "|");
    }

    @Override
    public void close() {
        pringLine();
    }

    private void pringLine() {
        System.out.print("+");
        for (int i = 0; i < width; i++) {
            System.out.print("-");
        }
        System.out.println("+");
    }
}
```

Main.java

```java
public class Main {
    public static void main(String[] args) {
        AbstractDisplay d1 = new CharDisplay('H');
        AbstractDisplay d2 = new StringDisplay("Hello,World.");
        d1.display();
        d2.display();
    }
}
```

运行结果

```console
<<HHHHH>>
+------------+
|Hello,World.|
|Hello,World.|
|Hello,World.|
|Hello,World.|
|Hello,World.|
+------------+
```

#### 相关的设计模式

- Factory Method模式
Factory Method模式是将Templete Method模式用于生成实例的一个典型例子。
  
- Strategy模式
在Templete Method模式中，可以**使用继承改变程序的行为**。这是因为Templete Method模式在父类中定义程序行为的框架，在子类中决定具体的处理。
  与此相对应的是Strategy模式，它可以**使用委托改变程序的行为**。与Templete Method模式中改变部分程序行为不同的是，Strategy模式用于替换整个算法。

#### 哪里用到了Templete Method模式

- `java.io.InputStream`类使用了Templete Method模式。

#### 参考资料

- 图解设计模式