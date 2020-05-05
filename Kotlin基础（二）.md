# Kotlin基础（二）

## 1、类和对象

与Java一样，Kotlin也是使用**class**关键字来声明一个类，实例化一个类的方式也是基本类似，只是去掉了**new**关键字。

例如，Person类的定义为：

```kotlin
class Person {
    var name = ""
}
```

其实例化代码为：

```kotlin
val person = Person()
```

## 2、继承与构造函数

#### 继承

不同于Java，默认情况下Kotlin的任何一个非抽象类都是**不可继承**的，其相当于Java中给类声明了**final**关键字。以Person类为例，要使其可以被继承，需要在类前声明**open**关键字。

```kotlin
open class Person {
    var name = ""
}
```

Kotlin中继承使用的关键字也不同于Java的**extends**，而是与C++一样使用冒号“**:**”。假设一个Student类继承于Person，写法为：

```kotlin
class Student : Person() {
    var id = ""
}
```

#### 构造函数

Student继承Person时后面为什么要加一对括号，就是因为Kotlin的构造函数。

Kotlin将构造函数分为两种：**主构造函数**和**次构造函数**。

##### 主构造函数

Kotlin中每个类默认都会有一个无参的主构造函数，当然也可以手动显式地指定参数。主构造函数的特点是没有函数体，直接定义在类名的后面。

以Student为例，其可改写为：

```kotlin
class Student(val id: String) : Person() {
}
```

此时实例化Student类时必须传入id参数。

对于需要在主构造函数中进行的类似初始化等逻辑，Kotlin提供了**init**结构体，相关逻辑可以写在里面。

```kotlin
class Student(val id: String) : Person() {
    init {
        println("id = " + id)
    }
}
```

根据Java继承规则，子类的构造函数中必须调用父类的构造函数，此规则Kotlin同样也要遵守。但是主构造函数没有函数体，所以Kotlin采用了与C++的构造类似的方法：括号。通过括号指定子类的构造函数调用父类的哪个构造函数。所以即使在无参的情况下，括号也不能省略。（当然也可以将调用父类构造的逻辑写在**init**中，但是**不推荐**这么做，因为一般情况下不需要**init**）。

以Person为例，将其改为：

```kotlin
open class Person(val name: String) {
}
```

Student则需要改为：

```kotlin
class Student(val id: String, name: String) : Person(name) {
}
```

需要注意，Student的主构造中name字段不能声明**val**或**var**，因为*主构造中声明成val或var的参数自动成为该类的字段*，这会导致与父类中的name字段冲突。

##### 次构造函数

Kotlin的任意一个类只能有**一个**主构造函数，但是可以有**多个**次构造函数。同时Kotlin规定，当一个类既有主构造函数又有次构造函数时，所有的次构造函数必须要调用主构造函数（包括间接调用）。

次构造函数通过**constructor**关键字定义。以Student为例：

```kotlin
class Student(val id: String, val grade: Int, name: String) : Person(name) {
    constructor(id: String, name: String) : this(id, 0, name) {}
    
    constructor() : this("", "") {}
}
```

**特殊情况**：Kotlin允许一个类没有主构造函数，只有次构造函数，如：

```kotlin
class Student : Person {
    constructor(name: String) : super(name) {}
}
```

此时，既然Student类没有主构造函数，继承Person类时也不需要指定父类构造函数，也就不需要加括号了。同时，由于自身没有主构造函数，且必须要调用父类构造函数，，所以是在次构造函数中直接通过super关键字调用父类主构造函数。

## 3、接口

与Java的单继承一样，Kotlin也通过接口实现多态性。

接口通过**interface**关键字定义，通过冒号“**:**”实现，当实现多个接口或者同时有继承和实现的时候，中间使用逗号“**,**”进行分隔。实现类或者子类中使用**override**关键字（不是注解）来实现接口中的或者重写父类的函数。

以Student为例，新建Study接口：

```kotlin
interface Study {
    fun readBooks()
    fun doHomework()
}
```

Student的实现为：

```kotlin
class Student(val id: String, name: String) : Person(name), Study {
    override fun readBooks() {
        println("readBooks")
    }
    
    override fun doHomework() {
        println("doHomework")
    }
}
```

**接口函数的默认实现**：Kotlin允许对接口中的函数进行默认实现。（同Java在JDK 1.8之后的接口一致）。

#### 函数可见性修饰符

与Java的4中函数修饰符**public**，**protected**，**private**，**default**（默认项）类似，Kotlin也提供4种修饰符：**public**（默认项），**protected**，**private**，**internal**。

- **private**的作用在Kotlin和Java中没有区别，都表示对当前类内部可见。

- **public**的作用也是一致的，表示对所有类可见，但是在Kotlin中它是默认项。

- **protected**在Java中表示对当前类、子类和同一包路径下的类可见，在Kotlin中表示只对当前类、子类可见。

- **internal**在Kotlin中表示对同一模块（如Android Studio中的Module等）中的类可见。Kotlin抛弃了Java的**default**修饰符，

## 4、数据类与单例类

#### 数据类

Java中的数据类一般格式为：

```java
public class Data {
    private int mValue;
    
    public int getValue() {return mValue;}
    
    public void setValue(int value) {this.mValue = value;}
    
    @Override
    public boolean equals(Object obj) {...}
    
    @Override
    public int hashCode() {...}
    
    @Override
    public String toString() {...}
}
```

在Kotlin则只需要在类前声明**data**关键字，就表示当前类是一个数据类，Kotlin会根据主构造函数中的参数自动生成equals()、hashcode()、toString()等固定但又没有实际逻辑意义的方法。

```kotlin
data class Data(val value: Int) {
    // 当类中没有任何代码时，可以省略类的大括号
}
```

#### 单例类

单例模式作为最常用的设计模式之一，Kotlin也对其进行了优化。

Java中的单例的一般形式为：

```java
public class Singleton {
    private static Singleton mInstance;
    
    private Singleton() {}
    
    public synchronized static Singleton getInstance() {
        if (mInstance == null) {
            mInstance = new Singleton();
        }
        return mInstance;
    }
    
    public void test() {}
}
```

Kotlin则只需要将类的声明由**class**改为**object**，一般形式如下：

```kotlin
object Singleton {
    fun test()
}
```

调用单例的函数时也不需要通过getInstance了，而是像调用静态方法一样直接调用`Singleton.test()`。因为Kotlin会在后台自动创建一个Singleton类的实例并保证全局唯一性。