*@Author 宾桀锋*  
*@Date 2017-11-02 09:58*

## java反射Reflection ##
### 一、基本介绍 ###
> 通过反射，我们可以在运行时获得程序或程序集中每一个类型的成员和成员的信息

程序中一般的对象的类型都是在编译期就确定下来的，而Java反射机制可以动态地创建对象并调用其属性，这样的对象的类型在编译期是未知的。所以我们可以通过反射机制直接创建对象，即使这个对象的类型在编译期是未知的。反射的核心是JVM在运行时才动态加载类或调用方法/访问属性，它不需要事先（写代码的时候或编译期）知道运行对象是谁。

Java反射框架主要提供以下功能：  
1. 在运行时判断任意一个对象所属的类；
2. 在运行时构造任意一个类的对象；
3. 在运行时判断任意一个类所具有的成员变量和方法（通过反射甚至可以调用private方法）；
4. 在运行时调用任意一个对象的方法

反射机制的作用：
1. 反编译：`.class`-->`.java`
2. 通过反射机制访问java对象的属性，方法，构造方法等

###### 1.已知类名 ######
```java
Class c1 = Student.class;
```
###### 2.已知对象 ###### 
```java
Class c2 = student.getClass();
```
c1、c2表示了Student类的类类型
类也是对象，是Class类的实例对象，这个对象称为该类的类类型。

###### 3.forName全类名 ###### 
```java
Class c3 = Class.forName("com.demo.Student");
```
**c1=c2=c3**

sun提供了哪些反射机制中的类：
```java
java.lang.Class; //类对象               
java.lang.reflect.Constructor; //类的构造器对象
java.lang.reflect.Field; //类的属性对象     
java.lang.reflect.Method; //类的方法对象
java.lang.reflect.Modifier;
```

### 二、使用 ###
**拿到了某个类的类类型，就拿到了该类的一切（方法、变量、构造函数等）**  
通过类的类类型创建该类的对象实例，如下面通过c1 or c2 or c3创建Student的实例：
```java
Student s = (Student)c1.newInstance();
```
（注：Student类必须要有一个无参数的构造方法）

**Class.forName("类的全称")**：
- 不仅表示了类的类类型，还代表了动态加载类
- 区分编译、运行
- 编译时刻加载类是静态加载类，运行时刻加载类是动态加载类

new创建对象是静态加载类，在编译时刻就需要加载所有的可能使用到的类。比如说，在java文件中，new一个不存在的类，编译就会报错。

```java
ArrayList list1 = new ArrayList();
ArrayList<String> list2 = new ArrayList<String>();

Class c1 = list1.getClass();
Class c2 = list2.getClass();

System.out.println(c1 == c2); //true
```
反射的操作都是编译之后的操作。c1==c2结果返回true说明编译之后集合的泛型是去泛型化的，java中集合的泛型，是防止错误输入的（如在eclipse中报错提示），只在编译阶段有效，绕过编译就无效了。验证：可以通过方法的反射来操作，绕过编译，如：
```java
Method m = c2.getMethod("add", Object.class); //list有add方法
m.invoke(list2, 100); //往ArrayList<String>中放入int类型的100，这是可以的
```

### 三、反射的用途 ###
当我们在使用IDE(如Eclipse，IDEA)时，当我们输入一个对象或类并想调用它的属性或方法时，一按点号，编译器就会自动列出它的属性或方法，这里就会用到反射。

**反射最重要的用途就是开发各种通用框架**

很多框架（比如Spring）都是配置化的（比如通过XML文件配置JavaBean,Action之类的），为了保证框架的通用性，它们可能需要根据配置文件加载不同的对象或类，调用不同的方法，这个时候就必须用到反射——运行时动态加载需要加载的对象。

举一个例子，在运用Struts2框架的开发中我们一般会在struts.xml里去配置Action，比如：
```xml
<action name="login" class="org.ScZyhSoft.test.action.SimpleLoginAction"method="execute">
    <result>/shop/shop-index.jsp</result>
    <result name="error">login.jsp</result>
</action>
```
配置文件与Action建立了一种映射关系，当View层发出请求时，请求会被StrutsPrepareAndExecuteFilter拦截，然后StrutsPrepareAndExecuteFilter会去动态地创建Action实例。比如我们请求login.action，那么StrutsPrepareAndExecuteFilter就会去解析struts.xml文件，检索action中name为login的Action，并根据class属性创建SimpleLoginAction实例，并用invoke方法来调用execute方法，这个过程离不开反射。