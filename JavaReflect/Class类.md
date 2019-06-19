# 【一】Class类

1. 在面向对象的世界中，万事万物都是对象；

2. 类也是对象，类是java.lang.Class类的实例对象；

3. Class实例对象的表示方式：（任何一个类都Class类的实例对象）

   * 任何一个类都有一个隐含的静态成员变量class

     ```java
     Class c1 = Foo.class
     ```

   * 已知该类的对象

     ```java
     Foo foo = new Foo()
     Class c2 = foo.getClass()
     c1 == c2 //true,都代表Foo类的类类型，一个类只可能是Class类的一个实例对象
     ```

   * Class.forName

     ```java
     Class c3 = Class.forName("com.xx.xx.Foo")//完整包路径
     ```

4. 通过类类型创建该类对象

```java
Foo foo = (Foo)c1.newInstance()//Foo需要有无参数构造方法
```

5. Class对象会在类编译时保存到.class文件，JVM运行时加载Class对象[（详解）](https://blog.csdn.net/javazejian/article/details/70768369)

