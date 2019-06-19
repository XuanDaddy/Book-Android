# 【五】AbstractProcessor详解

>  AbstractProcessor是javac扫描和处理注解的关键类。



1. init(ProcessingEnvironment env)
   * javac会在Processor创建时调用并执行的初始化操作，该方法会传入一个参数ProcessingEnvironment env，通过env可以访问Elements、Types、Filer等工具类。
   * Types env.getTypeUtils():返回一些用于对类型进行操作的实用方法实现； 
   * Filer env.getFiler():返回用于创建新的源，类及辅助文件的文件管理器； 
   * Messager env.getMessager():返回用于报告错误，警告和其他通知的工具；
   * Trees Trees.instance(env)：javac层面，Java树工具 

2. getSupportedAnnotationTypes()

   * 指定需要处理的注解类型；

     ```java
     Set<Class<? extends Annotation>> annotations = new LinkedHashSet<>();
     annotations.add(BindAnim.class);
     annotations.add(BindArray.class);
     annotations.add(BindBitmap.class);
     annotations.add(BindBool.class);
     annotations.add(BindColor.class);
     Set<String> types = new LinkedHashSet<>();
     for (Class<? extends Annotation> annotation : annotations) {
       types.add(annotation.getCanonicalName());
     }
     return types;
     ```

   * Class.getCanonicalName():大多数情况与getName()是一样的，完整限定名； 

   * Class.getCanonicalName()在数组和内部类中，与getName()有区别 

     ```java
     //内部类区别
     getName my.ExternalClassConfig$InternalConfig
     getCanonicalName my.ExternalClassConfig.InternalConfig
     getSimpleName InternalConfig
     //数组区别
     getName [Ljava.lang.String;
     getCanonicalName java.lang.String[]
     getSimpleName String]
     ```

3. getSupportedSourceVersion()

   * 指定支持的java版本，通常返回SourceVersion.latestSupported();
   * 如果只想支持到Java 7 可以返回 SourceVersion.RELEASE_7;
   * 默认返回SourceVersion.RELEASE_6;

4. process(Set<? extends TypeElement> annotations, RoundEnvironment env)

   * 它是每个 processor 的主方法，可以在这个方法中扫描和处理注解，并生成新的 java 源代码；

   * 通过参数 RoundEnvironment env 可以找到我们想要的某一个被注解的元素；

   * [Element](https://www.jianshu.com/p/a407a4901035) env.getElementsAnnotatedWith(BindView.class):返回使用给定注解类型注解的元素

     > **Element表示程序元素，如程序包，类或方法等。每个元素表示静态的语言级结构。详见《思维导图-Element》**
     >
     > **源代码的每一部分都是一个特定类型的Element；**
     >
     > 
     >
     > **子接口:**
     >
     > **TypeElement:一个类或接口元素；**
     >
     > **VariableElement:一个字段、enum常量、方法或构造方法参数、局部变量或异常参数；**
     >
     > **ExecutableElement:某个类或接口的方法、构造方法或初始化程序（静态或实例），包括注解类型元素；**
     >
     > **PackageElement:一个包程序元素；**
     >
     > **TypeParameterElement:一般类、接口、方法或构造方法元素的泛型参数；**

   * element.getAnnotation返回指定的注解类型

   * TypeElement element.getEnclosingElement()

     > **TypeElement表示一个类或接口程序元素。提供关于类型及其成员的信息访问。枚举类型是一种类，注释类型是一种接口。详见《思维导图-TypeElement》**

   * Name typeElement.getQualifiedName()

     ```java
     //Name表示一个字符序列，就是一种字符串。
     getQualifiedName():元素的完整限定名；
     getName():元素的简单名称；
     ```

   * Modifier element.getModifiers()

     ```java
     Modifier表示程序元素（如类、方法或字段）上的修饰符。
     ABSTRACT 修饰语 abstract
     DEFAULT 修饰语 default
     FINAL 修改者 final
     NATIVE 修饰语 native
     PRIVATE 修饰语 private
     PROTECTED 修饰语 protected
     PUBLIC 修饰语 public
     STATIC 修饰语 static
     STRICTFP 修饰语 strictfp
     SYNCHRONIZED 修饰语 synchronized
     TRANSIENT 修饰语 transient
     VOLATILE 修饰语 volatile
     ```

   * ElementKind element.getKind()

     ```java
     ElementKind表示一个元素的Kind。
     ANNOTATION_TYPE 注释类型。
     CLASS 没有一个更具体的类别描述的类（如 ENUM ）。
     CONSTRUCTOR 构造函数。
     ENUM 枚举类型。
     ENUM_CONSTANT 枚举常数。
     EXCEPTION_PARAMETER 异常处理程序的参数。
     FIELD 一个未被更具体种类描述的字段（如 ENUM_CONSTANT ）。
     INSTANCE_INIT 一个实例初始化器。
     INTERFACE 不具体 ANNOTATION_TYPE （如 ANNOTATION_TYPE ）。
     LOCAL_VARIABLE 一个局部变量。
     METHOD 方法。
     OTHER 一个实现保留元素。
     PACKAGE 包。
     PARAMETER 方法或构造函数的参数。
     RESOURCE_VARIABLE 资源变量。
     STATIC_INIT 静态初始化程序。
     TYPE_PARAMETER 一个类型参数。
     ```

   * TypeMirror element.asType()

     ```java
     TypeMirror表示Java编程语言中的一种类型。 类型包括原始类型，声明类型（类和接口类型），数组类型，类型变量和空类型。 还表示通配符类型参数，可执行文件的签名和返回类型以及对应于程序包和关键字void伪类型。
     直接子接口：
     ArrayType ，DeclaredType ，ErrorType ，ExecutableType ，IntersectionType ，NoType ， NullType ， PrimitiveType ，ReferenceType，TypeVariable ，UnionType ，WildcardType
     ```

   * TypeKind typeMirror.getKind()

     ```java
     TypeKind枚举，TypeMirror的可用范围。
     ARRAY 数组类型。
     BOOLEAN 原始类型 boolean。
     BYTE 原始类型 byte。
     CHAR 原始类型 char 。
     DECLARED 类或接口类型。
     DOUBLE 原始类型 double 。
     ERROR 无法解析的类或接口类型。
     EXECUTABLE 方法，构造函数或初始化器。
     FLOAT 原始类型 float 。
     INT 原始类型 int 。
     INTERSECTION 交叉路口型。
     LONG 原始类型 long 。
     NONE 在没有实际类型适合的情况下使用的伪类型。
     NULL 空类型。
     OTHER 实现保留类型。
     PACKAGE 对应于封装元件的伪类型。
     SHORT 原始类型为 short 。
     TYPEVAR 一个类型变量。
     UNION 联合类型。
     VOID 伪类型对应于关键字 void 。
     WILDCARD 通配符类型参数。
     ```

   * JCTree trees.getTree()

     > **JCTree涉及到javac的层面；**
     >
     > **TreeScanner可扫描Java树，根据 变量值 查找出 变量名**

   * Filer

     > **该接口支持通过注解处理器创建新文件。**
     >
     > **可创建类源文件及资源文件**