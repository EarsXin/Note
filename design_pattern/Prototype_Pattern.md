# 03 - 原型模式

用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。常用的 JSON.parseObject()也是一种原型模式。

原型模式主要运用在如下场景：

1. 类初始化资源较多
2. new 产生的一个对象需要非常繁琐的过程（数据准备、访问权限等）
3. 构造函数比较复杂
4. 循环体中产生大量的对象时

## 浅克隆

浅克隆一般是指简单的值类型数据复制，没有复制引用对象，复制之后的对象仍然指向原来的对象。

```java

public interface Prototype {
  Prototype clone();
}

public class ConcretePrototypeA implements Prototype {
  @Override
  public ConcretePrototypeA clone() {
    ConcretePrototypeA concretePrototype = new ConcretePrototypeA(); 
    concretePrototype.setAge(this.age); concretePrototype.setName(this.name); 
    concretePrototype.setHobbies(this.hobbies);
    return concretePrototype;
  }
}


public class Client {
  private Prototype prototype; public Client(Prototype prototype){
    this.prototype = prototype;
  }
  public Prototype startClone(Prototype concretePrototype){
    return (Prototype)  concretePrototype.clone();
  }
}
```

## 深克隆

深克隆解决了浅克隆引用对象指向的问题，使得克隆出来的对象是全新的对象实例

```java
public class Monkey {
    public int height;
    public int weight;
    public Date birthday;
}

public class JingGuBang implements Serializable {
    public float h = 100;
    public float d = 10;

    public void big() {
        this.d *= 2;
        this.h *=2;
    }

    public void small() {
        this.d /= 2;
        this.h /= 2;
    }
}

/**
* 齐天大圣类（克隆类），需要实现序列化和Cloneable接口
**/
public class QiTianDaSheng extends Monkey implements Cloneable, Serializable {
    public JingGuBang jingGuBang;

    public QiTianDaSheng() {
        this.birthday = new Date();
        this.jingGuBang = new JingGuBang();
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return this.deepClone();
    }

    public Object deepClone() {
        try {
            ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
            ObjectOutputStream objectOutputStream = new ObjectOutputStream(byteArrayOutputStream);
            objectOutputStream.writeObject(this);

            ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(byteArrayOutputStream.toByteArray());
            ObjectInputStream objectInputStream = new ObjectInputStream(byteArrayInputStream);

            QiTianDaSheng qiTianDaShengClone = (QiTianDaSheng) objectInputStream.readObject();
            qiTianDaShengClone.birthday = new Date();
            return  qiTianDaShengClone;


        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    public QiTianDaSheng shallowClone(QiTianDaSheng target) {
        QiTianDaSheng qiTianDaSheng = new QiTianDaSheng();
        qiTianDaSheng.birthday = target.birthday;
        qiTianDaSheng.height = target.height;
        qiTianDaSheng.weight = target.weight;

        qiTianDaSheng.jingGuBang = target.jingGuBang;

        qiTianDaSheng.birthday = new Date();

        return qiTianDaSheng;
    }
}

/**
* 深克隆测试类
**/
public class DeepCloneTest {
    public static void main(String[] args) {
        QiTianDaSheng qiTianDaSheng = new QiTianDaSheng();
        try {
            QiTianDaSheng clone = (QiTianDaSheng) qiTianDaSheng.clone();
            System.out.println("深克隆："+ (qiTianDaSheng.jingGuBang == clone.jingGuBang));
        }catch (Exception e) {
            e.printStackTrace();
        }

        QiTianDaSheng q = new QiTianDaSheng();
        QiTianDaSheng n = q.shallowClone(q);
        System.out.println("浅克隆：" + (q.jingGuBang == n.jingGuBang));
    }
}
```

## 小结

1. 原型模式说到底只是一种简单的创建对象的设计模式。在某些场景下确实能够减少代码。
2. 原型模式在一定程度上与单例模式相悖，如果做得不好会对单例模式产生破坏，一个比较推荐的做法是在单例类中实现 Cloneable 接口，在 clone 方法中返回单例对象，

```java
@Override
protected Object clone() throws CloneNotSupportedException {
  return INSTANCE;
}
```
