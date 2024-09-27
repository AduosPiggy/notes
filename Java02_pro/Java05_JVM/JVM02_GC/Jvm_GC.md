# JVM GC

## 1 如何判断对象可以回收

### 1.1引用计数法

定义：当一个对象A被饮用时，引用A的引用计数器 +1； 当引用失败或者结束时，对象A的引用计数器+1；当引用计数器 = 0时，说明A没有引用，可以被回收了。

缺陷：无法解决循环引用问题 --- 比如一堆垃圾互相的引用，如以下代码：

```Java
public class A {
    public static void main(String[] args) {
        TestA a = new TestA();
        TestB b = new TestB();
        a.b = b;
        b.a = a;
        a = null;
        b = null;
    }
}

class TestA {
    public TestB b;
}
class TestB {
    public TestA a;
}
```

### 1.2 可达性分析算法





### 1.3 引用

1.3.1 强引用

1.3.2 软引用 Soft Reference

1.3.3 弱引用 Weak Reference

1.3.4 虚引用 Phantom Reference

1.3.5 终结器引用 Final Reference

## 2 回收算法

2.1 标记清除法

2.2 标记整理法

2.3 复制算法

2.4 分代垃圾回收

## 3 垃圾回收器

## 4 垃圾回收调优