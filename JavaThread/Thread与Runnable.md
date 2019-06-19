# 【一】Thread与Runnable

#### 1、创建线程的两种方式

* Thread创建线程

```java
class Runner extends Thread {
    @Override
    public void run() {
        //...
    }
}
//启动线程
new Runner().start();
```

* Runnable创建线程

```java
class Runner implements Runnable {
    @Override
    public void run() {
        //...
    }
}
//启动线程
new Thread(new Runner()).start();
//直接使用匿名内部类
new Thread(new Runnable() {
    @Override
    public void run() {
        //...
    }
}).start();
```

#### 2、Thread与Runnable创建线程的区别

- Java是单继承模式，实现Runnable接口，还可以继承其它类；若直接继承Thread，就无法再继承自其它类；
- Thread每次都是新建一个类，Runnable可实现数据共享；
- 首选Runnable方式实现多线程；

