# Java8流库

## 从迭代到流的操作

* 流却能够以想要的任何方式来调度这些操作，只要结果是正确的即可
* 流与集合都可以转换和获取数据，但是他们之间存在差异
  * 流并不存储其元素，这些元素可能存储在底层的集合中，或者是按需生成
  * 流的操作不会修改其数据源
  * 流操作是尽可能惰性执行的，这意味着直到需要其结果时，操作才会执行，因此可以操作无限流
* 操作流的流程：
  * 创建一个流
  * 指定将初始流转换为其他流的中间操作，可能包含多个步骤
  * 应用终止操作，从而产生结果，这个操作会强制执行之前的惰性操作，从此之后，这个流就再也不能用了

## 流的创建

* 可以用Collection接口的stream方法将任意集合转换为一个流

* 可以通过Stream类的静态方法of，来将一个数组转换为一个流

* of方法具有可变长的参数，因此可以构建具有任意数量引元的流

* 使用Array.stream(array, from, to)可以用数组中的一部分元素来创建一个流

* 为了创建不包含任意元素的流，可以使用静态方法Stream.empty方法

* Stream接口有两个用于创建无限流的静态方法

  * generate方法会接受一个不包含任何引元的函数（或者从技术上讲，是一个Supplier<T>接口的对象）

  * 使用iterate方法，能够产生一个像0，1，2，3...这样的序列，它会接受一个种子值，以及一个函数（从技术上讲，是一个UnaryOperation<T>），并且会反复地将该函数应用到之前的结果上

    使用iterate方法产生一个有限序列，则需要添加一个谓词来描述迭代如何结束，只要该为此拒绝了迭代生成的值，这个流结束

    ```java
    BigInteger limit = new BigInteger("1000000");
    Stream<BigInteger> integers = Stream.iterate(BigInteger.ZERO, n -> n.compareTo(limit) > 0, n - > n.add(BigInteger.ONE))
    ```

* Stream.ofNullable方法会用一个对象来创建一个非常短的流，如果对象为null，那么这个流的长度就为0，否则，这个流的长度为1，只包含该对象

## filter、map和flatMap方法

* 流的装换会产生一个新的流，它的元素派生自另一个流中的元素

* filter的引元时Predicate<T>，从T到boolean的函数

* 按照某种方式来转换流中的值，此时，可以使用map方法并传递执行该转换的函数

* 将一个包含流的流摊平为单个流，可以使用flatMap方法而不是map方法

  ```java
  Stream<String> flatResult = words.stream().flatMap(w -> codePoints(w))
  ```

## 抽取子流和组合流

* 调用stream.limit(n)会返回一个新的流，它在n个元素之后结束（如果原来的流比n短，那么就会在该流结束时结束）
* 调用stream.skip(n)正好相反，它会丢弃前n个元素
* stream.takeWhile(predicate)调用会在谓词为真时获取流中的所有元素，然后停止
* dropWhile方法的做法正好相反，它会在条件为真时丢弃元素，产生一个由第一个使该条件为假的字符开始的所有元素构成的流
* 可以使用Stream类的静态方法concat方法将两个流连接起来

## 其他的流转换

* distinct方法会返回一个流，它的元素是从原有流中产生的，即原来的元素按照同样的顺序剔除重复元素后产生的
* 对于流的排序，有多种sorted方法的变体可用，其中一种用于操作Comparable元素的流，而另一种可以接受一个Comparator
* peek方法会产生另一个流，它的元素与原来流中的元素相同，但是在每次获取一个元素时，都会调用要给函数

## 简单约简

* 约简是一种终结操作，它们会将流约简为可以在程序中使用非流值
* count方法会返回流中元素的数量
* 其他简约还有max和min，他们分别返回最大值和最小值
* 这些方法返回的是一个类型Optional<T>的值，它要么其中包装了答案，要么表示没有任何值
* findFirst返回的是非空集合中的第一个值，它通常在与filter组合使用时很有用
* 如果不强调使用第一个匹配，而是使用任意的匹配都可以，那么就可以使用findAny方法，这个方法在并行流处理时很有效，因为流可以报告任何它找到的匹配而不是被限制为必须报告第一个匹配
* 如果想要知道是否存在匹配，那么可以使用anyMatch，这个方法会接受一个断言引元，因此不需要使用filter
* 还有allMatch和noneMatch方法，它们分别在所有元素和没有任何元素匹配谓词的情况下返回true

## Optional类型

### 获取Optional值

* 通常，在没有任何匹配时，我们希望使用某种默认值，可能是空字符串

  `String result = opetionalString.orElse("")`

* 还可以调用代码来计算默认值

  `String result = optionalString.orElseGet(() -> System.getProperty("myapp.default"))`

* 可以在没有任何值时抛出异常

  `String result = opetionalString.orElseThrow(IllegalStateException::new)`

### 消费Optional值

* ifPresent方法会接受一个函数，如果可选值存在，那么它会被传递给该函数，否则，不会发生任何事情
* 如果想要在可选值存在时执行一种动作，在可选值不存在时执行另一种动作，可以使用ifPresentOrElse

### 管道化Optional值

* 另一种有用的策略是保持Optional完整，使用map方法来转换Optional内部的值
* 可以使用filter方法来只处理那些在转换它你之前或之后满足某种特定属性的Optional值，如果不满足该属性，那么管道会产生空的结果
* 用or方法将空Optional替换为一个可替代的Optional，这个可替代值将以惰性方式计算

### 不适合使用Optional值得方式

* get方法会在Optional值存在的情况下获得其中包装的元素，或者在不存在的情况下抛出一个NoSuchElementException异常
* Optional类型正确的提示：
  * Optional类型的变量永远都不应该为null
  * 不要使用Optional类型的域，因为其代价是额外多出来一个对象，在类的内部，使用null表示确实的域更易于操作
  * 不要在集合中放置Optional对象，并且不要将它们用作map的键，应该直接收集其中的值

### 创建Optional值

* Optional.of(result)和Optional.empty()方法可以创建Optional值
* ofNullable方法被用来作为可能出现的null值和可选值之间的桥梁，Optional.ofNullable(obj)会在obj不为null的情况下返回Optional.of(obj)，否则返回Optional.empty()

### 用flatMap构建Optional值的函数

* flatMap方法，如果Optional存在，产生将mapper应用在当前Optional值所产生的结果，或者在当前Optional为空时，返回一个空Optional

### 将Optional转换为流

* stream方法会将一个Optional<T>对象转换为一个具有0ge或者1个元素的Stream<T>对象
* 每一个对stream的调用都会返回一个具有0个或1个元素的流，flatMap方法将这些方法组合在一起，这意味着不存在的会被直接丢弃

## 收集结果

* 调用iterator方法，它会产生用来访问元素的旧式风格的迭代器

* 可以调用forEach方法，将某个函数应用于每个元素

* 在并行流中，forEach方法会以任意顺序遍历各个元素，如果想要按照流中的顺序来处理，可以调用forEachOrdered方法，但是这个方法会丧失并行处理的部分甚至全部优势

* 通过调用toArray方法，可以获得由流的元素构成的数组

* 因为无法在运行时创建泛型数组，所以表达式会返回一个Object[]数组，如果想要让数组具有正确的类型，可以将其传递到数组构造器上

  `String[] result = stream.toArray(String[]::new)`

* 针对将流中的元素收集到另一个目标中，有一个便捷方法collect可用，她会接受一个collector接口实例

* 接收器是一种收集众多元素并产生单一结果的对象，collectors类提供了大量用于生成常见收集器的工厂方法

  * 要想将流的元素收集到一个列表中，应该使用Collectors.toList()方法产生的收集器

    `List<String> result = stream.collect(Collectors.toList())`

  * 将流收集到一个集中

    `Set<String> result = stream.collect(Collectors.toSet())`

  * 如果想要控制获得的集的种类

    `TreeSet<String> result = stream.collect(Collectors.toCollection(TreeSet::new))`

  * 通过连接操作来收集流中的所有字符串

    `String result = stream.collect(Collectors.joining())`

  * 如果想要在元素之间增加分隔符，可以将分隔符传递给joining方法

    `String result = stream.collect(Collectors.joinning(", "))`

* 想要将流的结果约简为总和、数量、平均值、最大值或最小值，可有使用summarizing（Int|Long|Double）方法中的某一个，这些方法接受一个将流对象映射为数值的函数，产生类型为（Int|Long|Double）SummaryStatistics的结果

## 收集到映射表中

* Collectors.toMap方法有两个函数引元，它们用来产生映射表的键和值
* 通常情况下，值应该时实际的元素，因此第二个函数可以使用Fucntion.identity()
* 如果有多个元素具有相同的键，就会存在冲突，收集器将会抛出一个IllegalStateException异常
* 可以通过提供第三个函数引元来覆盖这种行为，该函数会针对给定的已有值和新值来解决冲突并确定键对应的值
* 如果想要得到TreeMap，那么可以将构造器作为第四个引元来提供

## 群组和分区

* 将具有相同特性的值群聚成组时非常常见的，并且groupingBy方法直接支持它
* 当分类元素时断言函数时，流的元素可以分为两个列表，该函数返回true的元素和其他的元素
* 使用partitioningBy比使用groupingBy更高效

## 下游收集器

* groupingBy方法会产生一个映射表，它的每个值都是一个列表，如果想要以某种方式来处理这些列表，就需要提供一个“下游处理器”

  `Map<String, SetK<Local>> xxx = locales.collect(grupingBy(Locale::getCountry, toSet()))`

* Java提供了多种可以将收集到的元素约简为数字的收集器

  * counting会产生收集到的元素个数
  * Summing（Int|Long|Double）会接受一个函数作为引元，将该函数应用到下游元素中，并产生它们的和
  * maxBy和minBy会接受一个比较器，并分别产生下游元素中的最大值和最小值

* collectingAndThen收集器，在收集器后面添加了一个最终处理步骤

* mapping收集器的做法正好相反，它会将一个函数应用于收集到的每个元素，并将结果传递给下游收集器

* flatMapping方法，可以与返回流的函数一起使用

* 如果群组和映射函数的返回值为int、long或都变了，那么可以将怨怒收集到汇总统计对象中，然后，可以从每个组的汇总统计对象中获取这些函数值的总和、数量、平均值、最小值和最大值

* filtering收集器会将一个过滤器应用到每个组上

## 约简操作

* reduce方法是一种用于从流中计算某个值的通用机制，最简单的形式将接受一个二元函数，并从前两个元素开始持续应用它
* 如果流为空，那么该方法会返回一个Optional，因为没有任何有效的结果
* 通常，会有一个幺元值e使得e op x = x，可以使用这个元素作为计算的起点
* 如果流为空，则会返回幺元值，你就再也不需要处理Optional类

## 基本类型流

* 流库中具有专门的类型IntStream、LongStream、DoubleStream，用来直接存储基础类型值，而无需使用包装器
* 为了创建基本类型流，可以调用IntStream.of和Arrays.stream方法
* 与对象流一样，我们可以使用静态方法的generate和iterate方法
* 基本类型流有静态方法range和rangeClose，可以生成步长为1的整数范围
* 当你伊欧一个对象流时，可以用mapToInt、mapToLong或mapToDouble将其转换为基本类型流
* 为了将基本类型流转换为对象流可以使用boxed方法
* toArray方法返回基本类型数组
* 产生可选结果的方法会返回一个OptionalInt、OptionalLong或OptionalDouble，与Optional类似，但是具有getAsInt、getAsLong、getAsDouble方法
* 具有分别返回总和、平均值、最大值和最小值的方法
* summary Statistic会产生一个类型为IntSummaryStatistics、longxxx、doublexxx的对象，它们可以同时报告总和、数量、平均值、最大值和最小值

## 并行流

* 使用Collection.parallelStream()方法哦才能够任何集合中获取一个并行流
* parallel方法可以将任意的顺序六转换为并行流
* 只要在终结方法执行时流处于并行模式，所有的中间流操作就都将被并行化
* 当流操作并行运行时，其目标是让其返回结果与顺序执行时返回的结果相同
* 通过在流上调用Stream.unordered方法，就可以明确表示我们堆排序不感兴趣
* 还可以通过放弃排序要求来提高limit方法的速度
* 并行化会导致大量的开销，只有面对非常大的数据集才划算
* 只有在底层的数据源可以被有效的分割为多个部分时，将流并行化才由意义
* 并行流使用的线程池可能会因诸如文件I/O或网络访问这样的操作被阻塞而饿死

# 输入与输出

## 输入/输出流

* 在JavaAPI中，可以从其中读入一个字节序列额对象称作输入流，而可以向其中写入一个字节序列的对象称为输出流
* 抽象类InputStream和OutputStream构成了输入/输出类层次结构的基础
* 抽象类Reader和Writer中继承出来了一个专门用于处理Unicode字符的单独的类层次结构
* 这些类拥有的读入和写出操作都是基于两字节的char值得，而不是基于byte值得

### 读写字节

* InputStream类有一个抽象方法

  `abstract int read()`

* 这个方法将读入一个字节，并返回读入得字节，或者在遇到输入源结尾时返回-1

> * 从Java9开始，有了一个非常有用得可以读取流中所有字节得方法
>
>   `byte[] bytes = in.readAllBytes()`

* OutputStream类定义了一个抽象方法

  `abstrat void write(int b)`

* transferTo方法可以将所有字节从一个输入流传递到一个输出流

  `in.transferTo(out)`

* read和write方法在执行时都将阻塞，直至字节确实被读入或写出

* available方法是我们可以去检查当前可读入得字节数量

* 当你完成对输入/输出流得读写时，应该通过调用close方法来关闭它，这个调用会释放掉十分优先得操作系统资源

* close方法会关闭一个输出流的同时还会冲刷用于该输出流的缓冲区

* 还可以使用flush方法来人为地冲刷这些输出

### 完整的流家族

* 输入流与输出流的层次结构

  ![](D:\WorkSpace\Note\Java后端笔记\Java\输入流与输出流的层次结构.jpg)

* Reader和Writer的层次结构

  ![](D:\WorkSpace\Note\Java后端笔记\Java\Reader和Writer的层次结构.jpg)

* InputStream、OutputStream、Reader和Writer都实现了Closeable接口

* outputStream和Writer还实现了Flushable接口

* Readable接口只有一个方法

  `int read(CharBuffer cb)`

  CharBuffer类拥有按顺序和随机地进行读写访问的方法，它表示一个内存中的缓冲区或者一个内存映像的文件

* Appendable接口有两个用于添加单个字符和字符序列的方法

  `Appendable append(char c)`

  `Appendable append(CharSequence s)`

  CharSequence接口描述了一个char值序列的基础属性，String、CharBuffer、StringBuilder和String Buffer都实现了它

### 组合输入/输出流过滤器

* FileInputStream和FileOutputStream可以提供附着在一个磁盘文件上的输入流和输出流，而你只需向其构造器提供文件名或文件的完整路径名
* 某些输入流可以从文件和其他更外部的位置上获取字节，而其他的输入流可以将字节组装到更有用的数据结构中
* 可以通过嵌套过滤器来添加多重功能，相比之下，请求一个数据块并将其至于缓冲区中会显得更加高效
* 当读入输入时，可以预览下一个字节，Java提供了用于刺目的的PushbackInputStream
* Java必须将多个流过滤器组合起来，这种混合并匹配过滤器类以构建真正有用的输入/输出流序列的嗯呢力，将带来极大的灵活性

### 文本输入与输出

* OutputStreamWriter类将使用选定的字符编码方式，把Unicode码元的输出流转换为字节流，而InputStreamReader类将包含字节的输入流转换为可以产生Unicode码元的读入器
* 输入流读入其会假定使用主机系统所使用的默认字符编码方式，所以应该在InputStreamReader的构造器中选择一种具体的编码方式

### 如何写出文本输出

* 对于文本输出，可以使用PrintWriter，这个类拥有以文本格式打印字符串和数字的方法，为了打印文件，需要用文件名和字符编码方式构造一个PrintStream对象

  `PrintWriter out = new PrintWriter("xxx.txt", StandardCharsets.UTF_8)`

* 为了输出到打印写出器，需要使用与使用System.out相同的print、println和printf方法

* 如果写出器设置为自动冲刷模式，那么只要println被调用，缓冲区中的所有字符都会被发送倒塌的目的地

* 默认情况下，自动冲刷机制是禁用的，你要通过使用PrintWriter(Writer writer, booolean autoFlush)来启用或禁用自动冲刷机制

* print方法不抛出异常，你可以调用checkError方法来查看输出流是否出现了某些错误

### 如何读入文件输入

* 最简单的处理任何文本的方式就是使用Scanner类

* 想要将一个文件一行行地读入，那么可以调用

  `List<String> lines = Files.readAllLines(path, charset)`

* 如果文件太大，那么可以将行惰性处理为一个Stream<String>对象

  ```java
  try (Stream<String> lines = Files.lines(path, charset)) {
  	...
  }
  ```

* 还可以通过BufferedReader类进行文本输入处理，它的readLine方法会产生一行文本，或者在无法获得更多输入时返回null

* BufferedReader类又有一个lines方法，可以产生一个Stream<String>对象，但是BufferedReader没有用于任何读入数字的方法

### 字符编码方式

* 输入和输出流都是用于字节序列的，但是在许多情况下，我们希望操作的是文本

* Java针对字符使用的是Unicode标准

* 每个字符或编码点都具有一个21位的整数，有多种不同的字符编码方式，也就是说，将这些21位数字包装成字节的方式有多种

* 最常见的编码方式就是UTF-8，它会将每个Unicode编码点编码位1到4各字节的序列

* 另一个种常见的编码方式是UTF-16，它会将每个Unicode编码点编码为1个或两个16位值

* StandardCharsets类具有类型为Charset的静态变量，用于表示每种Java虚拟机都必须支持的字符编码方式

* 为了获取编码方式，可以使用静态方法forName方法

  `Charset shiftJIS = Charset.forName("Shift-JIS")`

## 读写二进制数据

### DataInput和DataOutput接口

* DataOutput接口定义了下面的以二进制格式写数组、字符、Boolean值和字符串的方法

  | writeChars | writeFloat   |
  | ---------- | ------------ |
  | writeByte  | writeDouble  |
  | writeInt   | writeChar    |
  | writeShort | writeBoolean |
  | writeLong  | writeUTF     |

* DataInput接口中定义的下列方法来读回数据

  | readInt   | readDouble  |
  | --------- | ----------- |
  | readShort | readChar    |
  | readLong  | readBoolean |
  | readFloat | readUTF     |

* DataInputStream类实现了DataInput接口，为了从文件中读入二进制数据看，可以将DataInputStream与某个字节源相组合

### 随机访问文件

* RandomAccessFile类可以在文件中的任何位置查找或写入数据
* 你可以打开一个随机访问文件，只用于读入或者同时用于读写，你可以通过使用字符串“r“或者”rw“作为构造器的第二个参数来指定这个选项
* 当你将已有文件作为RandomAccessFile打开时，这个文件并不会被删除
* 随机访问文件有一个表示下一个将被读入或写出的字节所处位置的文件指针，seek方法可以用来将这个文件指针设置到文件中的任意字节位置，seek的参数是一个long类型的整数，它的值位于0到文件按照字节来度量的长度之间
* getFilePointer方法返回文件指针的当前位置

### ZIP文档

* ZIP文档以压缩格式存储了一个或多个文件，每个ZIP文档都有一个头包含诸如每个文件名字和所使用的压缩方法等信息
* 在Java中，可以使用ZipInputStream来读入ZIP文档，你可能需要浏览文档中每个单独的项，getNextEntry方法就可以返回一个描述这些项的ZipEntry类型对象，该方法会从流中读入数据直至末尾，实际上这里的末尾是指正在读入的项的末尾，人或调用closeEntry来读入下一项，在读入最后一项之前，不要关闭zip
* 可以调用putNextEntry方法来写出新文件，并将文件数据发送daoZIP输出流中

## 对象输入/输出流与序列化

### 保存和加载序列化对象

* 为了保存对象数据，首先需要打开一个ObjectOutputStream独享

  `OutputStream out = new ObjectOutputStream(new FileOutputStream("xxx.dat"))`

* 为了保存对象i，可以直接使用ObjectOutputStream的writeObject方法

  `out.writeObject(xxx)`

* 为了将对象读回，首先需要获得一个ObjectInputStream对象

  `InputStream in = new ObjectInputStream(new FileInputStream("xxx.data"))`

* 然后调用readObject方法以这些对象被写出时的顺序获得他们

  `Object o = in.readObject()`

* 对希望在对象输出流中存储或从对象输入流中恢复的所有类都应进行一下修改，这个类必须实现Serilizable接口

* 当对象被重新加载时，他可能占据的是与原来完全不同的内存地址

* 每个对象都是用一个序列号保存的，这就是这个机制之所以称为对象寻猎话的原因

  * 对你遇到的每一个对象引用都关联一个序列号
  * 对于每个对象，当第一次遇到时，保存其对象那数据到输出流中
  * 如果某个对象之前已经被保存过，那么只写出“与之前保存过的序列号为X的对象相同”

* 读回对象时，整个过程是反过来的

  * 对于对象输入流中的对象，在第一次遇到其序列号时，构造它，并使用流中数据来初始化它，然后记录这个顺序号与新对象之间的关联
  * 当遇到“与之前保存过的序列号为x的对象相同”这一标记时，获取与这个序列号相关联的对象引用

## 操作文件

### Path

* Path表示的是一个目录名序列，其后还可以跟着一个文件名
* 路径中的第一个部件可以是根部件，而允许访问的根部件取决于文件系统
* 以根部件开始的路径时绝对路径，否则，就是相对路径
* 静态的Paths.get方法接受一个或多个字符串，并将它们用默认文件系统的路径分隔符连接起来
* 组合或解析路径是司空见惯的操作，调用path.resolve(x)将按照下列规则返回一个路径
  * 如果x是绝对路径，则结果就是x
  * 否则，根据文件系统的规则，将path后面跟着x作为结果

### 读写文件

* Files类可以使得普通文件操作变得快捷

  `byte[] bytes = Files.readAllBytes(path)`

* 如果希望将文件当作寒旭烈读入，那么可以调用

  `List<String> lines = Files.readAllLines(path, charset)`

* 希望写出一个字符串到文件中，可以调用

  `Files.writeString(path, content, charset)`

* 项指定文件追加内容，可以调用

  `Files.wirte(path, content.getBytes(charset), StandardOpenOption.APPEND)`

* 将一个行的集合写出到文件中

  `Files.write(path, lines,charset)`

### 创建文件和目录

* 创建新目录可以调用

  `Files.createDirectory(path)`

* 要求路径中除最后一个部件外，其他部分都必须是已存在的

* 要创建路径中的中间目录应该使用

  `Files.createDirectories(path)`

* 可以使用下面的语句创建一个空文件

  `Files.createFile(path)`

* 如果文件已经存在了，那么这个调用就会抛出异常，检查文件是否存在和创建文件是原子性的，如果文件不存在，该文件就会被创建，并且其他程序在此过程中是无法执行文件创建操作的

### 复制、移动和删除文件

* 将文件从一个位置复制到另一个位置可以直接调用

  `Files.copy(fromPath, toPath)`

* 移动文件可以调用

  `Files.move(fromPath, toPath)`

* 如果目标路径已经存在，那么复制或移动将失败。

* 如果想要覆盖已有的目标路径，可以使用REPLACE_EXISTING选项

* 如果想要复制所有问及那的属性，可以使用COPY_ATTRIBUTES选项

* 可以通过ATOMIC_MOVE选项来讲移动操作定义为原子性，这样就可以保证要么操作成功完成，要么源文件继续保持在原来的位置

* 删除文件可以调用

  `Files.delete(path)`

* 如果要删除的文件不存在，这个方法就会抛出异常，因此，可转而使用下面的方法

  `boolean deleted = Files.deleteIfExists(path)`

  该删除方法还可以用来移除空目录

* 用于文件操作的标准选项

  | 选项               | 描述                                                         |
  | ------------------ | ------------------------------------------------------------ |
  | StandardOpenOption | 与newBufferedWriter、newInputStream、newOutputStream、write一起使用 |
  | READ               | 用于读取而打开                                               |
  | WRITE              | 用于写入而打开                                               |
  | APPEND             | 如果用于写入而打开，那么在文件末尾追加                       |
  | TRUNCATE_EXISTING  | 如果用于写入而打开，那么移除已有内容                         |
  | CREATE_NEW         | 创建新文件并且在文件已存在情况下会创建失败                   |
  | CREATE             | 自动在文件不存在的情况下创建新文件                           |
  | DELETE_ON_CLOSE    | 当文件被关闭时，尽可能地删除该文件                           |
  | SPARSE             | 给文件系统一个提示，表示该文件是稀疏的                       |
  | DSYNC或SYNC        | 要求对文件数据\|数据和原数据的每次更新都必须同步地写入到存储设备中 |
  | StandardCopyOption | 与copy和move一起使用                                         |
  | ATOMIC_MOVE        | 原子性地移动文件                                             |
  | COPY_ATTRIBUTES    | 复制文件的属性                                               |
  | REPLACE_EXISTING   | 如果目标已存在，则替换他                                     |
  | LinkOption         | 与上面所有方法以及exists、isDirectory、isRegularFIle等一起使用 |
  | NOFOLLOW_LINKS     | 不要跟踪符号链接                                             |
  | FileVisitOption    | 与find、walk、walkFileTree一起使用                           |
  | FOLLOW_LINKS       | 跟踪符号链接                                                 |

### 获取文件信息

* 下面的静态方法都将返回一个boolean值，表示检查路径的某个属性结果

  * exists
  * isHidden
  * isReadable, isWritable, isExecutable
  * isRegularFile, isDirectory, isSymbolicLink

* size方法将返回文件的字节数

* getOwner方法将文件的拥有者作为一个实例返回

* 所有文件系统都会报告一个基本属性集，它们被封装在BasicFileAttributes接口中，这些属性与上述信息有部分重叠

* 基本文件属性包括：

  * 创建文件、最后一次访问以及最后一次修改文件的时间，这些时间都表示成java.nio.file.attribute.FileTime
  * 文件是常规文件、目录还是符号链接，异或这三者都不是
  * 文件尺寸
  * 文件主键，这是某种类对象，具体所属类与文件系统相关，有可能是文件的唯一标识符，也可能不是

* 获取文件属性可以调用

  `BasicFileAttributes attributes = Files.readAttributes(path, BasicFileAttributes.class)`;

### 访问目录中的项

* 静态的Files.list方法会返回一个可以读取目录中各个项的Stream<path>对象
* 目录是被惰性读取的，这使得处理具有大量项的目录可以变得更高效
* 为了处理目录中的所有子目录，需要使用File.walk方法
* 可以通过调用File.walk(pathToRoot, depth)来限制想要访问的树的深度

### 使用目录流

* 使用Files.newDirectoryStream对象，可以对遍历过程进行更加细粒度的控制，它会产生一个DirectoryStream

* 它是Iterable的子接口，以哦那次可以在增强的for循环中使用目录流

* 可以用glob模式来过滤文件

  | 模式  | 描述                                     |
  | ----- | ---------------------------------------- |
  | \*    | 匹配路径组成部分中0个或多个字符          |
  | \*\*  | 匹配跨目录边界的0个或多个字符            |
  | ?     | 匹配一个字符                             |
  | [...] | 匹配一个字符集合，可以使用连线符和取反符 |
  | {...} | 匹配由逗号隔开的多个可选项之一           |
  | \     | 转义上述任意模式中的字符以及\字符        |

* 如果想要访问某个目录的所有子孙成员，可以转而调用walkFileTree方法，并向其传递一个FileVisitor类型的对象，这个对象会得到下列通知

  * 在遇到一个文件或目录时

    `FileVisitResult visitFile(T path, BasicFileAttributes attrs)`

  * 在一个目录被处理前

    `FileVisitResult preVisitDirectory(T dir, IOException ex)`

  * 在一个目录被处理后

    `FileVisitResult postVisitDirecotry(T dir, IOException ex)`

  * 在视图访问文件或目录时发生错误

    `FileVisitResult visitFileFailed(path, IOEXception)`

  上述情况还可以指定是否希望执行下面操作

  * 继续访问下一个文件：FileVisitResult.CONTINUE
  * 继续访问，但是不在访问这个目录下的任何项：FileVisitResult.SKIP_SUBTREE
  * 继续访问，但是不在访问这个文件的兄弟文件：FileBisitResult.SKIP_SIBLINGS
  * 终止访问：FileVisitResult.TERMINATE

### ZIP文件系统

* Paths类会在默认文件系统中查找路径，也可以支持在其他文件系统中使用，其中最有用的之一就是ZIP文件系统

  `FileSystem fs = FileSystems.newFileSystem(Paths.get(zipName), null)`

* 要创建一个文件系统，它包含ZIP文档中的所有文件

* fs.getPath方法对于任意文件系统来说都是与Paths.get类似

## 内存映射文件

### 内存映射文件的性能

| 方法           | 时间 |
| -------------- | ---- |
| 普通输入流     | 110s |
| 带缓冲的输入流 | 9.9s |
| 随机访问文件   | 162s |
| 内存映射文件   | 7.2s |

* java.nio包使内存映射变得十分简单

  * 首先，从文件中获得一个通道，通道是用于磁盘文件的一种抽象，它使我们可以访问诸如内存映射、文件加锁机制以及文件间快速数据传递等操作系统特性

    `FileChannel channel = FileChannel.open(path, opetions)`

  * 然后，通过调用FileChannel类的map方法从这个通道中获得一个ByteBuffer

  * 你可以指定想要映射的文件区域与映射模式，支持的模式有三种

    * FileChannel.MapMode.READ_ONLY：所产生的缓冲区是只读的，任何对该缓冲区写入的尝试都会导致ReadOnlyBufferException异常
    * FileChannel.MapMode.READ_WRITE：所产生的缓冲区是可写的，任何修改都会在某个时候会到文件中，其他映射同一个文件的程序可能不能立即看到这些修改，多个程序同时进行文件映射的确切行为是依赖于操作系统的
    * FileChannel.MapMode.PRIVATE：所产生的缓冲区是可写的，但是任何修改对这个缓冲区来说都是私有的，不会传播到文件中

### 缓冲区数据结构

* 缓冲区是由具有相同类型的数值构成的数组，Buffer类是一个抽象类，它有众多的具体子类

* 最常用的将会是ByteBuffer和CharBuffer

  * 一个容量，他永远不会改变
  * 一个读写位置，下一个值将在此进行读写
  * 一个界限，超过它进行读写是没有意义的
  * 一个可选的标记，用于重复读入或写入操作

  ![](D:\WorkSpace\Note\Java后端笔记\Java\一个缓冲区.jpg)

* 调用flip方法将界限设置到当前位置，并把位置复位到0

* remaining方法返回正数时（他返回值是界限-位置）

* 调用get将缓冲区中所有的值都读入

* 调用clear方法使缓冲区为下一次写循环做好准备，clear方法将位置复位到0，并将界限复位到容量

* 如果想要重读缓冲区，可以使用rewind或mark/reset方法

* 想要获取缓冲区，可以调用ByteBuffer.allocate或ByteBuffer.wrap

## 文件加锁机制

* FileLock lock()——在整个文件上获得一个独占的锁，这个方法降阻塞直至获得锁
* FileLock tryLock()——在整个文件上获得要给独占的锁，或者在无法获得锁的情况下返回null
* void close()——释放这个锁

## 正则表达式

* 正则表达式用于指定字符串的模式
* 可以在任何需要定位匹配某种特定模式的字符串的情况下使用正则表达式

### 正则表达式语法

| 表达式                                                 | 描述                                                         |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| c, 除.\*+?{\|()[\\^$之外                               | 字符c                                                        |
| .                                                      | 任何除行终止符之外的字符，或者在DOTALL标志被设置时表示任何字符 |
| \\x{p}                                                 | 十六进制码为p的Unicode码点                                   |
| \uhhhh,\xhh,\0o,\0oo,\0ooo                             | 具有给定十六进制或八进制的码元                               |
| \a,\e,\f,\n,\r,\t                                      | 转义字符                                                     |
| \cc，其中c在[A-Z]的范围内或者时@[\\]^\_?之一           | 对应于字符c的控制字符                                        |
| \c，其中c不在[A-Za-z0-9]的范围                         | 字符c                                                        |
| \Q...\E                                                | 在左引号和右引号之间的所有字符                               |
| [C1C2...]，其中C1是多个字符，其范围从c-d，或者是字符类 | 任何由C1，C3，...表示的字符                                  |
| [^...]                                                 | 某个字符类的补集                                             |
| [...&&...]                                             | 字符集的交集                                                 |
| \p{...}，\P{...}                                       | 某个预定义字符类；它的补集                                   |
| \d，\D                                                 | 数字；它的补集                                               |
| \w，\W                                                 | 单词字符；它的补集                                           |
| \s，\S                                                 | 空格；它的补集                                               |
| \h，\v，\H，\V                                         | 水平空白字符、垂直空白字符；它们的补集                       |
| XY                                                     | 任何X中的字符串，后面跟追随任意Y中的字符串                   |
| X\|Y                                                   | 任何X或者Y中的字符串                                         |
| (X)                                                    | 捕获X的匹配                                                  |
| \n                                                     | 第n组                                                        |
| (?<name>X)                                             | 捕获于给定名字匹配的X                                        |
| \k<name>                                               | 具有给定名字的组                                             |
| (?:X)                                                  | 使用括号但是不捕获X                                          |
| X?                                                     | 可选X                                                        |
| X*，X+                                                 | 0或多个X，1或多个X                                           |
| X{n}，X{n,}，X{m, n}                                   | n个X，至少n个X，m到n个X                                      |
| ^，$                                                   | 输入的开头和结尾                                             |
| \A，\Z，\z                                             | 输入的开头、输入的结尾、输入的绝对结尾                       |
| \b，\B                                                 | 单词边界，非单词边界                                         |
| \R                                                     | Unicode行分隔符                                              |
| \G                                                     | 前一个匹配的结尾                                             |

* 大部分字符都可以与它们自身匹配
* .符号剋以匹配任何字符
* 使用\作为转义字符
* ^和$分别匹配一行的开头和结尾
* 如果X和Y是正则表达式，那么XY表示“任何X的匹配后面跟随Y的匹配”，X|Y表示“任何X或Y的匹配”
* 可以将量词运用到表达式X：X+、X*与X?
* 默认情况下，量词要匹配能够使整个匹配成功的最大可能的重复次数。可以修改这个默认行为，方法是使用后缀?（使用勉强或吝啬匹配，也就是匹配最小的重复次数）或者使用后缀+（使用占有或贪婪匹配，也就是及时让整个匹配失败，也要匹配最大的次数）
* 使用群组来定义子表达式，其中群组用括号()括起来

### 匹配字符串

* 首先用表示正则表达式的字符串构建一个Pattern对象，然后从这个表达式中获得一个Matcher，并调用它的matches方法

  ```java
  Pattern pattern = Pattern.compile(patternString):
  Matcher matcher = pattern.matcher(input)
  ```

* 在编译过程中，可以设置一个或多个标志

  ```java
  Pattern pattern = Pattern.compile(expression, 标志)
  ```

  * Pattern.CASE_INSENSITIVE或i：匹配字符时忽略字母的大小写，默认情况下，这个标志只考虑US ASCII字符
  * Pattern.UNICODE_CASE或u：当与CASE_INSENSITIVE组合使用时，用Unicode字母的大小写来匹配
  * Pattern.MULTILINE或m：^和$匹配行的开头和结尾，而不是整个输入的开头和结尾
  * Pattern.UNIX_LINES或d：在多行模式中匹配^和$时，只有‘\n’被识别成行终止符
  * Pattern.DOTALL或s：当使用这个标志时，.符号匹配所有字符，包括行终止符
  * Pattern.COMMENTS或x：空白字符和注释将被忽略
  * Pattern.CANON_EQ：考虑Unicode字符规范的等价性

  最后两个标志不能在正则表达式内部指定

* 如果想要在流中匹配元素，可以将模式转换为谓词

  ```
  Stream<String> strings = ...;
  Stream<String> result = strings.filter(pattern.asPredicate());
  ```

* 如果正则表达式包含群组，那么Matcher对象可以揭示群组的边界

  ```java
  int start(int groupIndex);
  int end(int groupIndex);
  ```

* 可以直接调用`String group(int groupIndex)`抽取匹配的字符串

### 找出多个匹配

* 想找出输入中一个或多个匹配的子字符串，这是可以使用Matcher类的find方法来查找匹配内容，如果返回true，在使用start和end方法来查找匹配内容，或者使用不带引元的group方法来获取匹配的字符串
* 可以调用results方法获取一个Stream<MatchResult>
* 如果处理的是文件中的数据，那么可以使用Scanner.findAll方法来获取一个Stream<MatchResult>，这样就无需先将内容读取到一个字符串中

### 用分隔符来分割

* 需要将输入按照匹配的分隔符断开，而其他部分保持不变，Pattern.split方法可以自动完成这项任务

* 如果有多个标记，那么可以惰性获取它们

  `Stream<String> tokens = xxx.splitAsStream(input)`

### 替换匹配

* Matcher类的replaceAll方法将正则表达式出现的所有地方都用替换字符串来替换
* replaceFirst方法将只替换模式的第一次出现

# 日期和时间API

## 日期线

* Java的Date和Time API规范要求Java使用的时间尺度为
  * 每天86400秒
  * 每天正午与官方时间精确匹配
  * 在其他时间点上，以精确定义的方式与官方时间接近匹配
* 静态方法调用Instant.now()会给出当前的时刻，可以使用equals和compareTo方法来比较两个Instant对象，因此可以将Instant对象用作时间戳
* Duration是两个时刻之间的时间量，你可以通过调用toNanos、toMillis、getSeconds、toMinutes、toHours和toDays来获得Duration按照传统单位度量的时间长度
* Instant和Duration类都是不可修改的类，所以其方法都会返回一个新的实例

## 本地日期

* LocalDate是带有年、月、日的日期，为了构建LocalDate对象，可以使用now或of静态方法
* 用于本地日期的等价物是Period，他表示的是流逝的年、月或日的数量

### 日期调整器

* TemporalAdjusters类提供了大量用于常见调整的静态方法
* 还可以通过实现TemporalAdjuster类来创建自己的调整器
* 使用with方法会返回一个新的LocalDate对象，而不会修改原来的对象，with加上调整器能够生成新的LocalDate
* lambda表达式的参数类型为Temporal，它必须被强制转型为LocalDate
* 可以用ofDateAdjuster方法来避免这种强制转型，该方法期望得到的参数是类型为UnaryOperator<LocalDate>的lambda表达式

## 本地时间

* LocalTime表示当日时刻，可以用now或of方法创建其实例
* API说明展示了常见的对本地时间的操作，plus和minus操作是按照一天24小时循环操作的
* LocalDateTime类表示日期和时间，这个类适合存储固定时区的时间点
* 如果计算需要跨越夏令时，或者需要处理不同失去的用户，那么可以使用ZonedDateTime

## 时区时间

* 给定一个时区Id，静态方法ZoneId.of(id)可以产生一个ZoneId对象
* 可以通过调用local.atZone(zoneId)用这个对象将LocalDateTime转换为ZonedDateTime对象，或者可以通过调用静态方法ZonedDateTime.of(year, month, day, hour, minute, second, nano, zoneId)来构造一个ZonedDateTime对象

## 格式化和解析

* DateTimeFormatter类提供了三种用于打印日期/时间值的格式器

  * 预定义的格式器
  * locale相关的格式器
  * 带有定制模式的格式器

* 预定义的格式器

  | 格式器                                                 | 描述                                                         |
  | ------------------------------------------------------ | ------------------------------------------------------------ |
  | BASIC_ISO_DATE                                         | 年、月、日、时区偏移量，中间没有分隔符                       |
  | ISO_LOCAL_DATE, ISO_LOCAL_TIME, ISO_LOCAL_DATE_TIME    | 分隔符为-、:、T                                              |
  | ISO_OFFSET_DATE, ISO_OFFSET_TIME, ISO_OFFSET_DATE_TIME | 类似ISO_LOCAL_XXX，但是有时区偏移量                          |
  | ISO_ZONED_DATE_TIME                                    | 有时区偏移量和时区ID                                         |
  | ISO_INSTANT                                            | 在UTC中，用z时区ID来表示                                     |
  | ISO_DATE, ISO_TIME, ISO_DATE_TIME                      | 类似ISO_OFFSET_DATE，ISO_OFFSET_TIME和ISO_ZONED_DATE_TIME，但是时区信息可选 |
  | ISO_ORDINAL_DATE                                       | LocalDate的年和年日期                                        |
  | ISO_WEEK_DATE                                          | LocalDate的年、星期和星期日期                                |
  | RFC_1123_DATE_TIME                                     | 用于邮件时间戳的标准，编纂于RFC822、并在RFC1123中将年份更新到4位 |

* 要使用标准的格式器，可以直接调用其format方法

  `String formatted = DateTimeFormatter.ISO_OFFSET_DATE_TIME.format(app)`

* 标准格式器主要是为了机器可读的使劲啊戳而设计，为了向人类读者表示日期和时间，可以使用Locale相关的格式器

  | 风格   | 日期                     | 时间           |
  | ------ | ------------------------ | -------------- |
  | SHORT  | 7/16/69                  | 9:32 AM        |
  | MEDIUM | Jul 16, 1969             | 9:32:00 AM     |
  | LONG   | July 16, 1969            | 9:32:00 AM EDT |
  | FULL   | Wednesday, July 16, 1969 | 9:32:00 AM EDT |

* 静态方法ofLocalizedDate、ofLocalizedTime和ofLocalizedDateTime可以创建这个格式器

  ```java
  DateTimeFormatter formatter = DateTimeFormatter.ofLocalizedDateTime(ForMatStyle.LONG);
  String formatted = formatter.format(app);
  ```

* 可以通过指定模式来定制自己的日期格式

  `formatter = DateTimeFormatter.ofPattern("xxx")`



