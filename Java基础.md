## Java基础

### Java 的三大特性：**封装、继承、多态**

Java 的三大特性是**面向对象编程 (OOP)** 的三大基石。它们分别解决了代码的**安全性、复用性、灵活性**问题，目的是让代码像现实世界的物体一样易于管理和扩展。

**面向对象编程 (Object-Oriented Programming, OOP)**：一种编程思路，把程序看作是多个互相协作的“物体”（对象），而不是一连串的指令。

### 解释深拷贝与浅拷贝，深拷贝的循环引用

拷贝（Copy）是指创建一个与原对象内容相同的副本。根据拷贝操作进入对象内部的深度，我们将其分为**浅拷贝**与**深拷贝**。

为了理解这两者，我们需要先明确两个特定名词：

- **栈内存（Stack）：** 存储基本数据类型（如数字、布尔值）和对象的**引用地址**（Reference）。
- **堆内存（Heap）：** 存储对象的**实际数据内容**。

浅拷贝：相当于让复制变量也指向了原来的对象。比如B复制A，那么B也指向了A所指向的对象。

**深拷贝：不仅复制对象本身，还复制对象内部的所有引用对象。** **相当于创建了一个新的对象。**

Java 中 `Object.clone()` 默认实现的是浅拷贝，如果需要深拷贝，需要在 `clone()` 方法中手动复制引用对象。

深拷贝的循环引用：

循环引用就是对象之间互相引用，形成一个环。

解决方法：需要用一个map，来记录已经复制过的对象

如果这个对象之前已经复制过了
就不要再复制一次
直接用之前复制好的那个对象

### Java 中 final 作用是什么？

用来修饰类、方法和变量。

修饰的类，不能被继承。

修饰的方法，不能被重写。

修饰基本数据类型的变量，变量的值不能被修改。

修饰引用类型的变量，变量的引用不能更改，但是对象本身的内容是可以进行修改的。

### Java 中 static的作用是什么？

static主要用来修饰类的成员（变量、方法、代码块）和内部类。可以这样想，用static修饰的类的成员，是跟类绑定的。

### 泛型

### 反射

本质 = **在运行时动态获取类信息并操作类**。操作Class元数据。

### 动态代理

本质 = 拦截方法调用，在方法执行前后插入代码

有两种：

#### 1、基于jdk的动态代理

这种是基于接口的动态代理，必须要有接口。

```java
public interface UserService { // 接口
    void save();
}

public class UserServiceImpl implements UserService { // 实现类

    public void save(){
        System.out.println("保存用户");
    }

}

UserService target = new UserServiceImpl(); // target = 真实对象


UserService proxy = // 创建代理对象
    (UserService) Proxy.newProxyInstance(
        target.getClass().getClassLoader(), // 类加载器
        target.getClass().getInterfaces(), // 代理类要实现的接口
        new InvocationHandler() { // 当代理对象的方法被调用时，执行invoke方法

            public Object invoke(Object proxy, // 当前代理对象
                                 Method method, // 当前调用的方法
                                 Object[] args) throws Throwable { // 方法参数

                System.out.println("before");

                Object result = method.invoke(target,args);

                System.out.println("after");

                return result;
            }
        }
    );


proxy.save();
/*
执行流程
proxy.save()
     ↓
invoke()
     ↓
before
     ↓
target.save()
     ↓
after
*/
```

#### 2、基于CGLIB的动态代理

CGLIB 的本质就是：运行时生成目标类的子类，通过：重写方法，实现：方法拦截

```java
class UserService {

    public void save(){
        System.out.println("保存用户");
    }
}

Enhancer enhancer = new Enhancer(); // 创建 CGLIB 代理生成器
enhancer.setSuperclass(UserService.class); // 代理类继承 UserService

enhancer.setCallback(new MethodInterceptor() { // MethodInterceptor的作用是拦截方法调用

    public Object intercept(Object obj, // 当前代理对象，代理类的实例
                            Method method, // 当前调用的方法
                            Object[] args, // 方法参数
                            MethodProxy proxy) throws Throwable { // 用于调用父类方法

        System.out.println("before");

        Object result = proxy.invokeSuper(obj, args);

        System.out.println("after");

        return result;
    }
});


UserService proxy = (UserService) enhancer.create(); // 创建代理对象，也就是生成UserService的子类的实例
proxy.save();

/*
执行流程
proxy.save()
   ↓
intercept()
   ↓
before
   ↓
invokeSuper()
   ↓
UserService.save()
   ↓
after
*/



```

