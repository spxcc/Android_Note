## Java反射

目录

- 0.先看个例子
- 1.基本概念
- 2.Class
  - 2.1 获取Class
  - 2.2 通过Class获取类修饰符和类型
- 3.Member
  - 3.1 Field
    - 3.1.1 获取Field
    - 3.1.2 获取变量类型、修饰符、注解
    - 3.1.3 获取、设置变量值
  - 3.2 Method
    - 3.2.1 获取Method
    - 3.2.2 获取方法返回类型
    - 3.2.3 获取方法参数类型
    - 3.2.4 获取方法声明抛出的异常类型
    - 3.2.5 获取方法参数名称
    - 3.2.6 通过反射调用方法
  - 3.3 Constructor
    - 3.3.1 获取构造方法
    - 3.3.2 创建对象
- 4.数组和枚举
  - 4.1 数组
  - 4.2 枚举
- 5.反射的缺点
- 参考

### 0. 先看个例子

```
public class People {
    private People(int age) {
    }
    public void show() {
        System.out.println("test reflect");
    }
}

//test code
Class<?> clazz = Class.forName("com.xfhy.ref.People");
Constructor<?> declaredConstructor = clazz.getDeclaredConstructor(int.class);
declaredConstructor.setAccessible(true);
People person = (People) declaredConstructor.newInstance(12);
person.show();
```

我通过一个简单的小栗子展示了Java中的反射能做到的事,People的构造方法是私有的,但是我们竟然能在代码里面通过这个构造方法构建出实例.妙啊

### 1. 基本概念

Java的反射机制能做到:

- 对于任意一个类,都能够知道这个类的所有属性和方法
- 对于任意一个对象,都能够调用它的任意一个方法和属性

这种动态获取的信息以及动态调用对象的方法的功能称为Java语言的反射机制.

反射机制主要涉及2个类: Class 和 Member.

### 2. Class

Class是反射能够实现的基础,Class是JDK提供的一个类,完整路径是`java.lang.Class`.本质上这个类和Math,String或者自己定义的各种类没有什么区别.

#### 2.1 获取Class

Java反射包java.lang.reflect中的所有类都没有public构造方法,想要获取这些类的实例,只能通过Class类获取,所以说如果想使用反射,必须先获取Class对象.

获取Class对象的几种方法:

1. Object.getClass() 对象实例调用getClass方法
2. The.class 类调用.class eg:String.class
3. Class.forName("") 传入类的全路径
4. The.TYPE eg:Double.TYPE
5. Class.getSuperclass() eg: 某个实例调用`getClass().getSuperclass()`

#### 2.2 通过Class获取类修饰符和类型

来看一下HashMap的类的声明

```
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>,Cloneable,Serializable{}
```

- `public` : 限定符
- `class` : Class关键字
- `<K,V>` : 泛型类型
- `HashMap` : 类名
- `AbstractMap` : 父类
- `Map<K,V>,Cloneable,Serializable` : 接口

下面就简单写个demo来演示一下如何获取这些信息

```
Class<?> clazz = HashMap.class;
//类名  java.util.HashMap
System.out.println("Class : " + clazz.getCanonicalName());
//类限定符 public
System.out.println(Modifier.toString(clazz.getModifiers()));
//类泛型信息  : K V
TypeVariable<? extends Class<?>>[] typeParameters = clazz.getTypeParameters();
if (typeParameters.length != 0) {
    StringBuilder stringBuilder = new StringBuilder("Parameters : ");
    for (TypeVariable<? extends Class<?>> typeParameter : typeParameters) {
        stringBuilder.append(typeParameter.getName()).append(" ");
    }
    System.out.println(stringBuilder.toString());
}

//获取类实现的所有接口  java.util.Map<K, V> interface java.lang.Cloneable interface java.io.Serializable
Type[] genericInterfaces = clazz.getGenericInterfaces();
if (genericInterfaces.length != 0) {
    StringBuilder interfaces = new StringBuilder("Implemented Interfaces : ");
    for (Type intf : genericInterfaces) {
        interfaces.append(intf.toString());
        interfaces.append(" ");
    }
    System.out.println(interfaces.toString());
}

//获取类的父类   java.util.AbstractMap
Class<?> superclass = clazz.getSuperclass();
if (superclass != null) {
    System.out.println(superclass.getCanonicalName());
}

//获取类的注解 很明显,只能获取RUNTIME类型的注解
Annotation[] annotations = clazz.getAnnotations();
if (annotations.length != 0) {
    StringBuilder annotation = new StringBuilder("Annotations : ");
    for (Annotation a : annotations) {
        annotation.append(a.toString());
        annotation.append(" ");
    }
    System.out.println(annotation.toString());
}
```

### 3. Member

Member有3个实现类:

1. java.lang.reflect.Field : 类变量
2. java.lang.reflect.Method : 类方法
3. java.lang.reflect.Constructor : 类构造方法

为了展示效果,先建个测试类:

```
public class People {

    private String name;
    @Deprecated
    public int age;

    public People(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void eat(String food) {
        System.out.println("eat food " + food);
    }

    public void eat(String... foods) {
        StringBuilder s = new StringBuilder();
        for (String food : foods) {
            s.append(food);
            s.append(" ");
        }
        System.out.println("eat food " + s.toString());
    }

    public void sleep() {
        System.out.println("sleep");
    }

    @Override
    public String toString() {
        return "name = " + name + " age = " + age;
    }
}
```

#### 3.1 Field

通过Field可以访问给定类对象的类变量,包括获取变量的类型,修饰符,注解,变量名,变量值等,即使是private的也是OK的.

##### 3.1.1 获取Field

Class提供了4种方法获得给定类的Field

- getDeclaredField(String name) 获取指定的变量,包括private的
- getField(String name) : 获取指定的变量,只能是public的
- getDeclaredFields() : 获取所有变量,包括private的
- getFields() : 获取所有变量,只能是public的

##### 3.1.2 获取变量类型、修饰符、注解

直接上代码,简单明了

```
Class clazz = People.class;
Field[] declaredFields = clazz.getDeclaredFields();
for (Field field : declaredFields) {
    StringBuilder stringBuilder = new StringBuilder();

    stringBuilder.append("变量名: ").append(field.getName()).append("\n");
    stringBuilder.append("变量类型: ").append(field.getType()).append("\n");
    stringBuilder.append("变量修饰符: ").append(Modifier.toString(field.getModifiers())).append("\n");
    Annotation[] annotations = field.getAnnotations();
    if (annotations.length != 0) {
        stringBuilder.append(" 变量注解 : ");
        for (Annotation a : annotations) {
            stringBuilder.append(a.toString());
            stringBuilder.append(" ");
        }
    }

    System.out.println(stringBuilder.toString());
}

//输出
变量名: name
变量类型: class java.lang.String
变量修饰符: private

变量名: age
变量类型: int
变量修饰符: public
变量注解 : @java.lang.Deprecated() 
```

##### 3.1.3 获取、设置变量值

给定一个对象和它的成员变量名称,就能通过反射获取和改变该变量的值.

```
People people = new People("张三", 17);
Class<? extends People> peopleClass = people.getClass();
try {
    //private的字段  需要使用getDeclaredField方法,且需要setAccessible为true
    Field nameField = peopleClass.getDeclaredField("name");
    nameField.setAccessible(true);

    Field ageField = peopleClass.getField("age");
    //反射 获取name字段
    String name = (String) nameField.get(people);
    int age = ageField.getInt(people);
    System.out.println("name = " + name + ", age = " + age);

    //设置成员变量的值
    nameField.set(people, "李四");
    ageField.set(people, 18);
    System.out.println(people.toString());

} catch (NoSuchFieldException | IllegalAccessException e) {
    e.printStackTrace();
}

//输出
name = 张三, age = 17
name = 李四 age = 18
```

需要注意的是private的Field获取出来之后,依然没有权限进行直接访问,Java运行时会进行访问权限检查.需要调用setAccessible为true才行,该方法的作用是可以取消Java语言访问权限检查.继承了AccessibleObject的类的对象(Field正是AccessibleObject的子类)都可以使用该方法取消Java语言访问权限检查.

> ps: Method和Constructor也都是继承AccessibleObject,遇到私有方法或私有构造方法无法访问时,处理方法也是一样的.

#### 3.2 Method

#### 3.2.1 获取Method

Class提供了4种方法获取Method

- `getDeclaredMethod(String name, Class<?>... parameterTypes)`
- `getMethod(Sting name, Class<?>... parameterTypes)`
- `getDeclaredMethods()`
- `getMethods()`

有2个需要注意的地方:

- 获取带参数方法时,如果参数类型错误会抛NoSuchMethodException
- 对于参数是泛型的情况下,泛型必须当成Object处理

#### 3.2.2 获取方法返回类型

- `getReturnType()` 获取目标方法返回类型对应的Class对象
- `getGenericReturnType()` 获取目标方法返回类型对应的Type对象

对于返回值是普通类型如Object、int、String等,两者返回值一样.但是有2种特殊情况:

```
//1. 返回值是泛型
public T function()
getReturnType() : class java.lang.Object
getGenericReturnType() : T

//2. 返回值为参数化类型
public Class<String> function()
getReturnType(): class java.lang.Class
getGenericReturnType(): java.lang.Class<java.lang.String>
```

#### 3.2.3 获取方法参数类型

- `getParameterTypes()` 获取目标方法各参数类型对应的Class对象
- `getGenericParameterTypes()` 获取目标方法各参数类型对应的Type对象

#### 3.2.4 获取方法声明抛出的异常类型

- `getExceptionTypes()` 获取目标方法抛出的异常类型对应的Class对象
- `getGenericExceptionTypes()` 获取目标方法抛出的异常类型对应的Type对象

#### 3.2.5 获取方法参数名称

Parameter的getName方法即可获取方法参数名称.

.class文件中默认不存储方法参数名称,如果想要获取方法参数名称,需要在编译的时候加上`-parameters`参数,构造方法的参数也是同样的.

#### 3.2.6 通过反射调用方法

通过Method的invoke()方法来反射调用目标方法,第一个参数为需要调用的目标类对象.如果方法是static的,则该参数为null,后面的参数是目标方法的参数值,顺序与目标方法声明中的参数顺序一致.

注意: 被调用的方法本身所抛出的异常在反射中都会以InvocationTargetException抛出. 反射调用过程中如果InvocationTargetException被抛出,说明反射调用本身是成功的,因为这个异常是目标方法本身所抛出的异常.

#### 3.3 Constructor

通过反射访问构造方法并通过构造方法构建新的对象

##### 3.3.1 获取构造方法

和Method类似

- `getDeclaredConstructor(Class<?>... parameterTypes)`
- `getConstructor(Class<?>... parameterTypes)`
- `getDeclaredConstructors()`
- `getConstructors()`

##### 3.3.2 创建对象

一般使用`java.lang.reflect.Constructor.newInstance()`来构建对象,另一种`Class.newInstance()`已经废弃.

注意: 反射不支持自动装箱,传入参数时需要小心.自动装箱是在编译期间的,而反射是在运行期间.

### 4. 数组和枚举

在反射中,对数组和枚举这两个对象的创建、访问和普通对象不同,Java反射为数组和枚举提供了特定的API接口.

#### 4.1 数组

- 数组类型

数组本质上是一个对象,所以它也有自己的类型.例如对于`int[] intArray`,数组类型为`class [I`.数组类中的`[`个数代表数组的维度,例如`[`代表一维数组,`[[`代表二维数组,`[`后面的字母代表数组元素类型,`I`代表`int`,一般为类型的首字母大写(long类型例外,为J.L被引用对象类型占用了)

- 创建和初始化数组

Java反射为我们提供了java.lang.reflect.Array 类用来创建和初始化数组

```
Array.newInstance(Class<?> componentType, int... dimensions)
Array.set(Object array, int index, int value)
Array.get(Object array, int index)
```

举个例子:

```
//用反射创建 int[] array = new int[]{1, 2}
Object array = Array.newInstance(int.class, 2);
Array.set(array, 0, 1);
Array.set(array, 1, 2);
System.out.println(Array.get(array, 1));
```

- 多维数组

Java反射没有提供能够直接访问多维数组元素的API,但你可以把多维数组当成数组的数组处理

```
Object matrix = Array.newInstance(int.class, 2, 2);
Object row1 = Array.get(matrix, 0);
Object row2 = Array.get(matrix, 1);
Array.set(row1, 0, 1);
Array.set(row1, 1, 2);
Array.set(row2, 0, 3);
Array.set(row2, 1, 4);
```

#### 4.2 枚举

枚举隐式继承自java.lang.Enum,Enum继承自Object.所以枚举本质也是一个类,也可以有成员变量,构造方法,方法等.对于普通类能使用的反射方法,枚举都能使用.另外Java反射额外提供了几个方法为枚举服务.

- `Class.isEnum()` 是否为枚举类型
- `Class.getEnumConstants()` 顺序检索枚举定义的枚举常量列表
- `java.lang.reflect.Field.isEnumConstant()` 判断此字段是否为枚举类型的元素

### 5. 反射的缺点

- 性能开销

反射涉及类型动态解析,所以JVM无法对这些代码进行优化.因此,反射操作的效率要比那些非反射操作低得多.我们应该避免在经常被执行的代码或对性能要求很高的程序中使用反射.

- 安全限制

使用反射要求程序在一个没有安全限制的环境中运行.如果一个程序必须在有安全限制的环境中运行,如Applet,那么这就是个问题了.

- 内部曝光

由于反射允许代码执行一些在正常情况下不被允许的操作,所以使用反射可能会导致意料之外的副作用

### 参考

- [Editing Android-Notes/反射.md at master · xfhy/Android-Notes (github.com)](https://github.com/xfhy/Android-Notes/edit/master/Blogs/Java/基础/反射.md)
- [Java反射完全解析](https://www.jianshu.com/p/607ff4e79a13)