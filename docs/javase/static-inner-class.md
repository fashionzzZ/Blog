
``` java
/**
 * 外部类
 **/
public class Outer {
    public static String staticField = "外部类-静态变量";

    public String field = "外部类-实例变量";

    {
        System.out.println("外部类-普通代码块");
    }

    static {
        System.out.println("外部类-静态代码块");
    }

    public Outer() {
        System.out.println("外部类-构造函数");
    }

    public void methord() {
        System.out.println("外部类-普通方法");
    }

    public static void staticMethord() {
        System.out.println("外部类-静态方法");
    }

    /**
     * 非静态内部类
     */
    class Inner {
        public String field = "非静态内部类-实例变量";

        {
            System.out.println("非静态内部类-普通代码块");
        }

        Inner() {
            System.out.println("非静态内部类-构造函数");
        }

        public void methord() {
            System.out.println("非静态内部类-普通方法");
        }
    }

    /**
     * 静态内部类
     */
    static class StaticInner {
        public static String staticField = "静态内部类-静态变量";

        public String field = "静态内部类-实例变量";

        {
            System.out.println("静态内部类-普通代码块");
        }

        static {
            System.out.println("静态内部类-静态代码块");
        }

        StaticInner() {
            System.out.println("静态内部类-构造函数");
        }

        public void methord() {
            System.out.println("静态内部类-普通方法");
        }

        public static void staticMethrod() {
            System.out.println("静态内部类-静态方法");
        }
    }
}
```

``` java
/**
 * 调用类
 **/
public class Main {
    public static void main(String[] args) {
        System.out.println("【外部类】");
        System.out.println("<--- 1.直接调用外部类的静态方法 --->");
        Outer.staticMethord();
        System.out.println("<--- 1.创建外部类对象，调用普通方法 --->");
        Outer outer = new Outer();
        outer.methord();

        System.out.println();

        System.out.println("【非静态内部类】");
        System.out.println("<--- Java不允许非静态内部类定义静态成员 --->");
        System.out.println("<--- 创建非静态内部类对象，调用普通方法 --->");
        Outer.Inner inner = outer.new Inner();
        inner.methord();

        System.out.println();

        System.out.println("【静态内部类】");
        System.out.println("<--- 1.直接调用静态内部类的静态方法 --->");
        Outer.StaticInner.staticMethrod();
        System.out.println("<--- 2.创建静态内部类对象，调用普通方法 --->");
        Outer.StaticInner staticInner = new Outer.StaticInner();
        staticInner.methord();
    }
}
```

### 内部类

内部类是定义在另外一个类中的类

- 内部类方法可以访问该类定义所在的作用域中的数据，包括私有的数据。
- 内部类可以对同一个包的其他类隐藏

#### 内部类的特殊语法规则

如果非静态内部类方法访问某个变量

1. 该方法中：`直接用该变量名`
2. 内部类中：`使用this.变量名`
3. 外部类中：`使用外部类的类名.this.变量名`

### 普通成员和静态成员的区别

普通成员：属于这个类的对象，程序运行后生成堆栈中。

静态成员：属于这个类，数据存放在class文件中，程序运行之前就已经存进去数据了。

### 非静态内部类和静态内部类的区别

`非静态内部类：必须依赖外部类实例，不允许有静态成员，必须用外部类实例来创建。`

`静态内部类：不依赖于外部类实例，可以有静态成员，可以理解为独立的一个类，只是写在了另外一个类中。`

`非静态内部类编译后隐式保存着外部类的引用（就算外部类对象没用了也GC不掉），但是静态内部类没有。`



#### 参考资料

- [内部类与静态内部类](https://www.cnblogs.com/GrimMjx/p/10105626.html)