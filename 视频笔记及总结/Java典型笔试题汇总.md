# <center>Java常见笔试题总结

#### 1、考察静态块、继承执行顺序

```java
public class Base extends BaseClass{
    public Base() {
        System.out.println("I'm Base class");
    }
    static {
        System.out.println("static Base");
    }

    public static void main(String[] args) {
        new Base();
    }
}

class BaseClass {
    public BaseClass() {
        System.out.println("I'm BaseClass class");
    }
    static {
        System.out.println("static BaseClass");
    }
}
```

输出内容：

```
static BaseClass
static Base
I'm BaseClass class
I'm Base class
```

