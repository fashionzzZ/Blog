### 结论

`静态变量和静态代码块优先于实例变量和普通代码块，静态变量和静态代码块的初始化顺序取决于它们在代码中的顺序。`

> 父类（静态变量、静态代码块）
>
> 子类（静态变量、静态代码块）
>
> 父类（实例变量、普通代码块）
>
> 父类（构造函数）
>
> 子类（实例变量、普通代码块）
>
> 子类（构造函数）


### 子类继承父类

#### 父类

这里我把父类声明为抽象类，并声明一个抽象方法abstractMethord()方便子类去覆盖。

``` java
/**
 * 父类
 **/
public abstract class AbstractFather {
    public static String staticField = "父类-静态变量";

    public String field = "父类-实例变量";

    {
        System.out.println("父类-普通代码块");
    }

    static {
        System.out.println("父类-静态代码块");
    }

    AbstractFather() {
        System.out.println("父类-构造函数");
    }

    public void fatherMethord() {
        System.out.println("父类-普通方法");
    }

    /**
     * 抽象方法
     */
    public abstract void abstractMethord();
}
```

#### 子类

``` java
/**
 * 子类
 **/
public class Son extends AbstractFather {
    public static String staticField = "子类-静态变量";

    public String field = "子类-实例变量";

    {
        System.out.println("子类-普通代码块");
    }

    static {
        System.out.println("子类-静态代码块");
    }

    Son() {
        System.out.println("子类-构造函数");
    }

    public void sonMethord() {
        System.out.println("子类-普通方法");
    }

    @Override
    public void abstractMethord() {
        System.out.println("子类-覆盖父类的抽象方法");
    }
}
```

#### Main方法调用

``` java
public class Main {
    public static void main(String[] args) {
        AbstractFather son = new Son();
        son.fatherMethord();
        son.abstractMethord();
    }
}
```

控制台打印：

``` console
父类-静态变量
父类-静态代码块
子类-静态变量
子类-静态代码块
父类-实例变量
父类-普通代码块
父类-构造函数
子类-实例变量
子类-普通代码块
子类-构造函数
父类-普通方法
子类-覆盖父类的抽象方法
```

