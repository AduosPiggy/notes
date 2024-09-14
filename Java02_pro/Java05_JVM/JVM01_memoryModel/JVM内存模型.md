# JVM内存模型

致敬作者

| 作者名称 | 参考地址                                                    | 参考详情      |
| -------- | ----------------------------------------------------------- | ------------- |
| hmb↑     | https://blog.csdn.net/qq_48721706/article/details/128170423 | JVM内存模型图 |

JVM内存模型如下：

<img src="./assets/01_JvmModel.png" alt="01_JvmModel" style="zoom: 50%;" />

## 1  程序计数器 PC Register

### 1.1 定义

- 作用: 	记住下一条jvm指令的执行地址
- 特点:
  - 是线程私有的
  - 不会存在内存溢出

### 1.2 plus

- 解释器会解释指令为机器码交给 cpu 执行，程序计数器会记录下一条指令的地址行号，这样下一次解释器会从程序计数器拿到指令然后进行解释执行。
- 多线程的环境下，如果两个线程发生了上下文切换，那么程序计数器会记录线程下一行指令的地址行号，以便于接着往下执行。

## 2 虚拟机栈 JVM Stack

### 2.1 定义

- 每个线程运行时所需要的内存，称为虚拟机栈
- 每个栈由多个栈帧（Frame）组成，对应着每次方法调用时所占用的内存
- 每个线程只能有一个活动栈帧，对应着当前正在执行的那个方法

### 2.2 理解

- **线程请求的栈深度大于虚拟机所允许的深度，抛出StackOverflowError；**
- **虚拟机栈扩展时无法申请到足够的内存，抛出OutOfMemoryError。**

### 2.3 StackOverflowError

```Java
/**
 * 栈超出最大深度：StackOverflowError
 * VM args: -Xss128k
 **/
public class StackSOF {
    private int stackLength = 1;
    public void stackLeak(){
        stackLength++;
        stackLeak();
    }
    public static void main(String[] args) {
        StackSOF stackSOF = new StackSOF();
        try {
            stackSOF.stackLeak();
        } catch (Throwable e) {
            System.out.println("当前栈深度:" + stackSOF.stackLength);
            e.printStackTrace();
        }
    }
}
```

```Java
java.lang.StackOverflowError
当前栈深度:30170
at cn.itcast.jvm.StackSOF.stackLeak(StackSOF.java:11)
at cn.itcast.jvm.StackSOF.stackLeak(StackSOF.java:11)

方法递归调用，造成深度过深，产生异常
```

### 2.4 OutOfMemoryError

(谨慎使用，会卡死电脑)

```Java
/**
 * 栈内存溢出： OOM
 * VM Args: -Xss2m
 **/
public class StackOOM {
    private void dontStop(){
        while (true){
        }
    }
    public void stackLeakByThread(){
        while(true){
            Thread t = new Thread(new Runnable() {
                public void run() {
                    dontStop();
                }
            });
            t.start();
        }
    }
    public static void main(String[] args) {
        StackOOM stackOOM = new StackOOM();
        stackOOM.stackLeakByThread();
    }
}
```

```Java
Exception in thread "main"  java.lang.OutOfMemoryError:unable to create new native thread
```

### 2.5 辨析

| 问题                         | 答案                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| 1 垃圾回收是否设计栈内存？   | 不会，栈帧内在每次方法结束后都会弹出栈，自动回收掉           |
| 2 占内存分配越大越好？       | 不对，因为物理内存是一定的，栈内存越大，可以支持更多的递归调用，但是可执行的线程数目就会越少。 |
| 3 方法内的局部变量是否安全？ | 若是采用了引用对象，并且能够逃离方法的作用范围，则需要考虑线程安全 |

针对问题3案例：

```Java
//线程安全, 引用未能逃离方法m1
public void m1(){
    StringBuilder sb = new StringBuilder();
    sb.append("1")
            .append("2")
            .append("3");
}
//需考虑线程安全, sb为引用, 逃离m2控制
public void m2(StringBuilder sb){
    sb.append("1")
            .append("2")
            .append("3");
}
//需考虑线程安全, sb为引用, 逃离m3控制
public StringBuilder m3(){
    StringBuilder sb = new StringBuilder();
    sb.append("1")
            .append("2")
            .append("3");
    return sb;
}
```

## 3 本地方法栈 Native Method Stack

## 4 堆 Heap

## 5 方法区 Method Area

## 6 直接内存