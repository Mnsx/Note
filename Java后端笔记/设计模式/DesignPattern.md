# 设计模式

## 学习资料

* 图解设计模式（人民邮电出版社）

## Iterator模式

**Iterator模式用于在数据集合中按照顺序遍历集合**

### 案例参考

书籍实体类

```java
public class Book {
    private String name;
    public Book(String name) {
        this.name = name;
    }
    public String getName() {
        return name;
    }
}
```

所要遍历的集合的接口

```java
public interface Aggregate {
    public abstract Iterator iterator();
}
```

所要遍历的集合接口的实现类

```java
public class BookShelf implements Aggregate {
    private Book[] books;
    private int last = 0;
    public BookShelf(int maxSize) {
        this.books = new Book[maxSize];
    }
    public Book getBookAt(int index) {
        return books[index];
    }
    public void appendBook(Book book) {
        this.books[last] = book;
        last++;
    }
    public int getLength() {
        return last;
    }
    @Override
    public Iterator iterator() {
        return new BookShelfIterator(this);
    }
}
```

用于遍历集合中元素的接口

```java
public interface Iterator {
    public abstract boolean hasNext();
    public abstract Object next();
}
```

Iterator接口的实现类

```java
public class BookShelfIterator implements Iterator {
    private BookShelf bookShelf;
    private int index;

    public BookShelfIterator(BookShelf bookShelf) {
        this.bookShelf = bookShelf;
        this.index = 0;
    }

    @Override
    public boolean hasNext() {
        if (index < bookShelf.getLength()) {
            return true;
        } else {
            return false;
        }
    }

    @Override
    public Object next() {
        Book book = bookShelf.getBookAt(index);
        index++;
        return book;
    }
}
```

Main类

```java
public class Main {
    public static void main(String[] args) {
        BookShelf bookShelf = new BookShelf(4);
        bookShelf.appendBook(new Book("demo1"));
        bookShelf.appendBook(new Book("demo2"));
        bookShelf.appendBook(new Book("demo3"));
        bookShelf.appendBook(new Book("demo4"));
        Iterator iterator = bookShelf.iterator();
        while (iterator.hasNext()) {
            Book book = (Book)iterator.next();
            System.out.println(book.getName());
        }
    }
}
```

### 抽象类和接口

* 抽象类需要使用继承实现，很容易造成类之间的强耦合
* 接口能够软化类之间的耦合，进而使得类更加容易作为组件被再次利用

### Iterator的“下一个”和“最后一个”

next方法是**返回当前的元素，并指向下一个元素**

hasNext方法理解成**确认接下来是否可以调用next方法**

## Adapter模式

**Adapter模式用于填补”现有的程序“和”所需的程序“之间差异的设计模式**

### 案例参考

#### 类适配器模式（使用继承的适配器）

现有的程序

```java
public class Banner {
    private String string;

    public Banner(String string) {
        this.string = string;
    }

    public void showWithParen() {
        System.out.println("(" + string + ")");
    }

    public void showWithAster() {
        System.out.println("*" + string + "*");
    }
}
```

适配器接口

```java
public interface Print {
    void printWeak();
    void printStrong();
}
```

所需的程序

```java
public class PrintBanner extends Banner implements Print {
    public PrintBanner(String string) {
        super(string);
    }

    @Override
    public void printWeak() {
        showWithParen();
    }

    @Override
    public void printStrong() {
        showWithAster();
    }
}
```

Main类

```java
public class Main {
    public static void main(String[] args) {
        Print p = new PrintBanner("Hello");
        p.printWeak();
        p.printStrong();
    }
}
```

#### 对象适配器模式（使用委托的适配器）

现有的程序

```java
public class Banner {
    private String string;

    public Banner(String string) {
        this.string = string;
    }

    public void showWithParen() {
        System.out.println("(" + string + ")");
    }

    public void showWithAster() {
        System.out.println("*" + string + "*");
    }
}
```

适配器委托程序（抽象类）

```java
public abstract class Print {
    public abstract void printWeak();
    public abstract void printStrong();
}
```

所需的程序

```java
public class PrintBanner {
    private Banner banner;

    public PrintBanner(String string) {
        this.banner = new Banner(string);
    }

    public void printWeak() {
        banner.showWithParen();
    }

    public void printStrong() {
        banner.showWithAster();
    }
}
```

Main类

```java
public class Main {
    public static void main(String[] args) {
        Print p = new PrintBanner("Hello");
        p.printWeak();
        p.printStrong();
    }
}
```

### Adapter使用环境

1. 使用Adapter模式，可以在不改变源代码的情况下，使现有代码适配新的接口
2. 代码版本升级并且能够兼容旧版本

## Template Method模式

**Template Method模式使带有模板功能的模式，组成模板的方法被定义在父类中（抽象方法）**

**在父类中定义处理流程的框架，在子类中实现具体处理**

### 案例参考

抽象父类模板

```java
public abstract class AbstractDisplay {
    public abstract void open();
    public abstract void print();
    public abstract void close();
    public final void display() {
        open();
        for (int i = 0; i < 5; ++i) {
            print();
        }
        close();
    }
}
```

按照模板的实体子类

```java
public class CharDisplay extends AbstractDisplay {
    private char ch;

    public CharDisplay(char ch) {
        this.ch = ch;
    }

    @Override
    public void open() {
        System.out.print("<<");
    }

    @Override
    public void print() {
        System.out.print(ch);
    }

    @Override
    public void close() {
        System.out.println(">>");
    }
}
```

```java
public class StringDisplay extends AbstractDisplay {
    private String string;
    private int width;

    public StringDisplay(String string) {
        this.string = string;
        this.width = string.getBytes(StandardCharsets.UTF_8).length;
    }

    public void printLine() {
        System.out.print("+");
        for (int i = 0; i < width; ++i) {
            System.out.print("-");
        }
        System.out.println("+");
    }

    @Override
    public void open() {
        printLine();
    }

    @Override
    public void print() {
        System.out.println("|" + string + "|");
    }

    @Override
    public void close() {
        printLine();
    }
}
```

Main类

```java
public class Main {
    public static void main(String[] args) {
        AbstractDisplay d1 = new CharDisplay('H');
        AbstractDisplay d2 = new StringDisplay("Hello world");
        AbstractDisplay d3 = new StringDisplay("你好 世界");

        d1.display();
        d2.display();
        d3.display();
    }
}
```

### Template Method模式优点

由于在父类的模板方法中编写了算法，因此无需在每个子类中再编写算法

### 父类与子类之间的协作

父类和子类是紧密联系、共同工作的。因此，在子类中实现父类中声明的抽象方法时，必须要理解这些抽象方法的调用时机

## Factory Method模式

### 案例参考

工厂模板类

```java
public abstract class Factory {
    public final Product create(String owner) {
        Product p = createProduct(owner);
        registerProduct(p);
        return p;
    }

    protected abstract Product createProduct(String owner);
    protected abstract void registerProduct(Product product);
}
```

工厂类产品模板类

```java
public abstract class Product {
    public abstract void use();
}
```

工厂模板生成类

```java
public class IDCardFactory extends Factory {
    private List owners = new ArrayList();

    @Override
    protected Product createProduct(String owner) {
        return new IDCard(owner);
    }

    @Override
    protected void registerProduct(Product product) {
        owners.add(((IDCard)product).getOwner());
    }

    public List getOwners() {
        return owners;
    }
}
```

工厂类产品模板生成类

```java
public class IDCard extends Product {
    private String owner;

    IDCard(String owner) {
        this.owner = owner;
    }

    @Override
    public void use() {
        System.out.println("使用" + owner + "的ID卡");
    }

    public String getOwner() {
        return owner;
    }
}
```

Main类

```java
public class Main {
    public static void main(String[] args) {
        Factory factory = new IDCardFactory();
        Product card1 = factory.create("xkq");
        Product card2 = factory.create("ysy");
        Product card3 = factory.create("xyl");
        card1.use();
        card2.use();
        card3.use();
    }
}
```

### 框架与具体加工

在框架中不会引用加工类，因此，即使用已有的框架生成全新的类时，也完全不需要对framework进行修改

### 生成实例——方法的三种实现方式

* 指定其为抽象方法

  指定其为抽象方法，讲构建方法指定为抽象方法后，子类就必须实现该方法，如果子类不实现该方法，编译器将会报告编译错误

* 为其实现默认处理

  为其实现默认处理。实现默认处理后，如果子类没有实现该方法，将进行默认处理

* 在其中抛出异常

  在其中抛出异常的方法。如果未在子类中实现方法，程序将会在运行时出错

  不过需要另外编写FactoryMethodRuntimeException异常类

## Singleton模式

### 案例参考

单例类

```java
public class Singleton {
    private static Singleton singleton = new Singleton();

    private Singleton() {
        System.out.println("创建了一个实例");
    }

    public static Singleton getInstance() {
        return singleton;
    }
}
```

Main类

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Start .");
        Singleton obj1 = Singleton.getInstance();
        Singleton obj2 = Singleton.getInstance();
        if (obj1 == obj2) {
            System.out.println("obj1与obj2是相同的实例");
        } else {
            System.out.println("obj1与obj2是不同的实例");
        }
    }
}
```

### 唯一实例的生成时机

程序运行后，在第一次调用getInstance方法时，Singleton类会被初始化。也就是在这个时候，static字段singleton被初始化，生成了唯一的一个实例