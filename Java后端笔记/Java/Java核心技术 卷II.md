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