# 03 - 单例模式

单例模式（Singleton Pattern）是指确保一个类在任何情况下都绝对只有一个实例，并且提供全局访问点。单例模式是创建性模式。经典案例： 在 Spring 框架应用中 ApplicationContext;数据库的连接池也都是单例形式

## 恶汉式单例

饿汉式单例是在类加载的时候就立即完成初始化，并且创建单例对象，这种方式在线程还没有出现以前就实例化了，能够做到绝对的线程安全，不可能出现访问安全问题。

```java
// 通过类静态成员属性的加载顺序进行单例对象的初始化
public class HungrySingleton {
    private static final HungrySingleton instance = new HungrySingleton();
    private HungrySingleton() {}
    public static HungrySingleton getInstance() {
        return instance;
    }
}

// 通过静态代码块的加载顺序对单例对象进行对象初始化
public class HungryStaticSingleton {
    private static final HungryStaticSingleton instance;
    static {
        instance = new HungryStaticSingleton();
    }
    private HungryStaticSingleton() {}
    public static HungryStaticSingleton getInstance() {
        return instance;
    }
}

```

优点： 没有加上任何的锁，执行效率比较高，在用户体验上，比懒汉式更好
缺点： 类加载的时候就进行初始化，会出现这个初始化的对象没有使用的情况，浪费类内存空间

## 懒汉式单例模式

前面介绍了饿汉式单例模式，我们能够发现它有优点和缺点，如果我们想要解决掉它的缺点呢？这就产生了蓝函数单例模式，被外部内调用的时候才会进行加载

```java
//懒汉式单例 //在外部需要使用的时候才进行实例化
public class LazySimpleSingleton {
  private LazySimpleSingleton(){} //静态块，公共内存区域
  private static LazySimpleSingleton lazy = null;
  public static LazySimpleSingleton getInstance(){

  if(lazy == null){
    lazy = new LazySimpleSingleton();
  }

  return lazy; }
}
```

这种代码写出来之后我们在使用 ExectorThread 的时候，会出现安全隐患（创建多个不同的单例对象），我们想到了给 getInstance 方法加上 synchronized 关键字进行约束，将 getInstance 方法编程同步方法。在线程数量比较多的情况下如果 CPU 分配压力上升，会导致大量线程堵塞，从而导致程序运行性能大幅下降。有能够实现能够兼容线程安全和性能的实现方式 -- 双重检查锁的单例模式和静态类内部类实现懒汉式单例模式

```java

/**
* 双重检查锁懒汉式单例模式
**/
public class LazyDoubleCheckSingleton {
    // volative 解决指令重排序
    private volatile static LazyDoubleCheckSingleton lazy = null;
    private LazyDoubleCheckSingleton() {
    }

    /**
     * 双重检查锁
     * @return
     */
    public static LazyDoubleCheckSingleton getInstance() {
        if (lazy == null) { // 保证线程能够进入
            synchronized (LazyDoubleCheckSingleton.class) {
                if(lazy == null) {  // 避免多次重复创建被覆盖
                    lazy = new LazyDoubleCheckSingleton();
                    // CPU执行时就会转换成JVM指令执行
                    // 1. 分配内存给对象
                    // 2. 初始化对象
                    // 3. 将初始化的对象和内存地址建立关联
                    // 4. 用户初次访问

                    // 第 2,3步  执行顺序可能会打乱，出现指令重排序
                }
            }
        }
        return lazy;
    }
}


/**
* 静态内部类实现懒汉式单例模式
**/
public class LazyInnerClassSingleton {
    private LazyInnerClassSingleton() {
        // 这里解决反射的方式构造单例的实例，解决不走寻常路的反射攻击方式
        if (LazyHolder.LAZY != null) {
            throw new RuntimeException("不允许构建多个实例");
        }
    }

    // 懒汉式单例
    // LazyHolder 需要等到外部的方法
    // JVM 底层的实现逻辑，完美避免线程安全问题
    public static final LazyInnerClassSingleton getInstance() {
        return LazyHolder.LAZY;
    }

private static class LazyHolder{
    private static final LazyInnerClassSingleton LAZY = new LazyInnerClassSingleton();
}

    // 重写readResolve方法，只不过是覆盖了反序列化出来的对象
    // 最终还是创建了两次，但是发生在JVM底层，相对来说还是比较安全的
    // 之前反序列化的结果将会被GC进行回收
    private Object readResolve() {
        return LazyHolder.LAZY;
    }
}

```

相信肯定注意到了代码中有两出比较奇怪的地方了：

1. `LazyInnerClassSingleton`: 中对反射创建单例对象的处理了，这是因为在 java 中可以通过反射绕过正常的创建方式，所以对于不走寻常路的常见方式采用了直接抛出错误的处理方式了，
2. `readResolve`: 当我们将一个单例对象创建好，有时候需要将对象序列化然后写入到磁盘，下次使用时 再从磁盘中读取到对象，反序列化转化为内存对象。反序列化后的对象会重新分配内存， 即重新创建。那如果序列化的目标的对象为单例对象，就违背了单例模式的初衷，相当 于破坏了单例。在 jdk 底层源码实现上 ObjectInputStream 类的 readObject()方法中又调用了开发者重写的 readObject() 方法

## 注册式单例

注册式单例有称之为登记是单例，将每一个实例都登记到某一个地方，使用唯一的标志的标志进行实例获取。注册式单例有两种写法：容器缓存和枚举登记

### 枚举式到哪里

```java
// 枚举式到哪里
public enum EnumSingleton {
  INSTANCE;
  private Object data; public Object getData() {
    return data;
  }
  public void setData(Object data) {
    this.data = data;
  }
  public static EnumSingleton getInstance(){
    return INSTANCE;
  }
}
```

> 枚举式单例在静态代码块中共就给 ISNSTANCE 进行赋值，是饿汉式单例的实现
> 枚举类型其实就是通过类名和 class 对象找到唯一的枚举对象，不可能出现呗类加载多次的情况
> Constructor#newInstance()方法做了强制性的判断，如果是修饰符 Modifier.ENUM 枚举类型，直接抛出异常

### 容器式单例

```java
public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory implements AutowireCapableBeanFactory {
  /** Cache of unfinished FactoryBean instances: FactoryBean name --> BeanWrapper */
  private final Map<String, BeanWrapper> factoryBeanInstanceCache = new ConcurrentHashMap<>(16);
  ...
}
```

容器式写法适用于创建实例非常多的情况，便于管理。但是，是非线程安全的，这一点我们可以重 Spring 容器式单例可以看出

## ThreadLocal 线程单例

ThreadLocal 不能保证其 创建的对象是全局唯一，但是能保证在单个线程中是唯一的，天生的线程安全，利用这种特性能够实现数据源安全的切换

```java
public class ThreadLocalSingleton {
  private static final ThreadLocal<ThreadLocalSingleton> threadLocalInstance =new ThreadLocal<ThreadLocalSingleton>(){
    @Override
    protected ThreadLocalSingleton initialValue() {
      return new ThreadLocalSingleton();
    }
  };
  private ThreadLocalSingleton(){}
  public static ThreadLocalSingleton getInstance(){
    return threadLocalInstance.get();
  }
}
```

## 小结

单例模式可以保证内存里只有一个实例，减少了内存开销;可以避免对资源的多重占用。 单例模式看起来非常简单，实现起来其实也非常简单。
