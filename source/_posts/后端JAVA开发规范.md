---
title: 后端JAVA开发规范
date: 2022-0118 13:25:00
author: EverSpring
top: false
toc: false
mathjax: false
categories: JAVA
tags:
  - Java
  - 规范
---
根据原公司规范、阿里巴巴JAVA规范形成

## 后端JAVA开发规范
JAVA开发规范一节借鉴了《阿里巴巴JAVA开发手册》，根据公司情况做了一些删减、补充，建议大家在网上下载《阿里巴巴JAVA开发手册》当前最新版本为嵩山版（截止2022.1.18）仔细阅读，官方地址：https://github.com/alibaba/p3c 
建议大家安装代码检查工具，比如：Alibaba Java Coding Guidelines、SpotBugs、SonarLint等，开发完成后做代码扫描
#### 一、 命名
1. 【强制】类名使用 UpperCamelCase 风格，但以下情形例外：DO / BO / DTO / VO / AO / PO / UID 等
正例：ForceCode / UserDO / HtmlDTO / XmlService / TcpUdpDeal / TaPromotion
反例：forcecode / UserDo / HTMLDto / XMLService / TCPUDPDeal / TAPromotion
2. 【强制】方法名、参数名、成员变量、局部变量都统一使用 lowerCamelCase 风格。
正例： localValue / getHttpMessage() / inputUserId
3. 【强制】常量命名全部大写，单词间用下划线隔开，力求语义表达完整清楚，不要嫌名字长。
正例：MAX_STOCK_COUNT / CACHE_EXPIRED_TIME
反例：MAX_COUNT / EXPIRED_TIME
4. 【强制】抽象类命名使用 Abstract 或 Base 开头；异常类命名使用 Exception 结尾；测试类命名以它要测试的类的名称开始，以 Test 结尾
5. 【强制】类型与中括号紧挨相连来表示数组。
正例：定义整形数组 int[] arrayDemo。
反例：在 main 参数中，使用 String args[]来定义
6. 【强制】POJO 类中的任何布尔类型的变量，都不要加 is 前缀，否则部分框架解析会引起序列化错误
7. 【强制】包名统一使用小写，点分隔符之间有且仅有一个自然语义的英语单词。包名统一使用单数形式，但是类名如果有复数含义，类名可以使用复数形式
正例：应用工具类包名为 com.alibaba.ei.kunlun.aap.util、类名为 MessageUtils（此规则参考 spring 的框架结构）
8. 【强制】避免在子父类的成员变量之间、或者不同代码块的局部变量之间采用完全相同的命名，使可理解性降低。说明：子类、父类成员变量名相同，即使是 public 类型的变量也能够通过编译，另外，局部变量在同一方法内的不同代码块中同名也是合法的，这些情况都要避免。对于非 setter/getter 的参数名称也要避免与成员变量名称相同。
9. 【参考】枚举类名带上 Enum 后缀，枚举成员名称需要全大写，单词间用下划线隔开
说明：枚举其实就是特殊的常量类，且构造方法被默认强制是私有
正例：枚举名字为 ProcessStatusEnum 的成员名称：SUCCESS / UNKNOWN_REASON

#### 二、 OOP规约
1. 【强制】避免通过一个类的对象引用访问此类的静态变量或静态方法，无谓增加编译器解析成本，直接用类名来访问即可
2. 【强制】所有的覆写方法，必须加@Override 注解
说明：getObject()与 get0bject()的问题。一个是字母的 O，一个是数字的 0，加@Override 可以准确判断是否覆盖成功。另外，如果在抽象类中对方法签名进行修改，其实现类会马上编译报错
3. 【强制】相同参数类型，相同业务含义，才可以使用 Java 的可变参数，避免使用 Object
说明：可变参数必须放置在参数列表的最后（建议开发者尽量不用可变参数编程）
正例：public List<User> listUsers(String type, Long... ids) {...}
4.  【强制】Object 的 equals 方法容易抛空指针异常，应使用常量或确定有值的对象来调用 equals
正例："test".equals(object)
反例：object.equals("test");
说明：推荐使用 JDK7 引入的工具类 java.util.Objects#equals(Object a, Object b)
5.  【强制】所有整型包装类对象之间值的比较，全部使用 equals 方法比较
6.  【强制】任何货币金额，均以最小货币单位且整型类型来进行存
7.  【强制】浮点数之间的等值判断，基本数据类型不能用==来比较，包装数据类型不能用 equals来判断
8. 【强制】浮点数之间的等值判断，基本数据类型不能用==来比较，包装数据类型不能用 equals来判断
说明：equals()方法会比较值和精度（1.0 与 1.00 返回结果为 false），而 compareTo()则会忽略精度
9. 【强制】定义数据对象 DO 类时，属性类型要与数据库字段类型相匹配
正例：数据库字段的 bigint 必须与类属性的 Long 类型相对应
反例：某个案例的数据库表 id 字段定义类型 bigint unsigned，实际类对象属性为 Integer，随着 id 越来越大，超过 Integer 的表示范围而溢出成为负数
10.  【强制】禁止使用构造方法 BigDecimal(double)的方式把 double 值转化为 BigDecimal 对象
说明：BigDecimal(double)存在精度损失风险，在精确计算或值比较的场景中可能会导致业务逻辑异常。
如：BigDecimal g = new BigDecimal(0.1F); 实际的存储值为：0.10000000149
正例：优先推荐入参为 String 的构造方法，或使用 BigDecimal 的 valueOf 方法，此方法内部其实执行了Double 的 toString，而 Double 的 toString 按 double 的实际能表达的精度对尾数进行了截断。
```
BigDecimal recommend1 = new BigDecimal("0.1");
BigDecimal recommend2 = BigDecimal.valueOf(0.1);
```
11. 关于基本数据类型与包装数据类型的使用标准如下
	* 【强制】所有的 POJO 类属性必须使用包装数据类型
	* 【强制】RPC 方法的返回值和参数必须使用包装数据类型
	正例：数据库的查询结果可能是 null，因为自动拆箱，用基本数据类型接收有 NPE 风险
12.  【强制】序列化类新增属性时，请不要修改 serialVersionUID 字段，避免反序列失败；如果完全不兼容升级，避免反序列化混乱，那么请修改 serialVersionUID 值。
说明：注意 serialVersionUID 不一致会抛出序列化运行时异常
13.   【强制】构造方法里面禁止加入任何业务逻辑，如果有初始化逻辑，请放在 init 方法中
14.   【强制】禁止在 POJO 类中，同时存在对应属性 xxx 的 isXxx()和 getXxx()方法
说明：框架在调用属性 xxx 的提取方法时，并不能确定哪个方法一定是被优先调用到的
15.   【推荐】使用索引访问用 String 的 split 方法得到的数组时，需做最后一个分隔符后有无内容的检查，否则会有抛 IndexOutOfBoundsException 的风险
16.   【推荐】慎用 Object 的 clone 方法来拷贝对象
说明：对象 clone 方法默认是浅拷贝，若想实现深拷贝，需覆写 clone 方法实现域对象的深度遍历式拷贝

#### 三、日期时间
1.  【强制】日期格式化时，传入 pattern 中表示年份统一使用小写的 y
说明：日期格式化时，yyyy 表示当天所在的年，而大写的 YYYY 代表是 week in which year（JDK7 之后 引入的概念），意思是当天所在的周属于的年份，一周从周日开始，周六结束，只要本周跨年，返回的 YYYY 就是下一年
2.   【强制】获取当前毫秒数：System.currentTimeMillis(); 而不是 new Date().getTime()
3.   【强制】不允许在程序任何地方中使用：1）java.sql.Date。 2）java.sql.Time；3）java.sql.Timestamp
说明：第 1 个不记录时间，getHours()抛出异常；第 2 个不记录日期，getYear()抛出异常；第 3 个在构造方法 super((time/1000)*1000)，在 Timestamp 属性 fastTime 和 nanos 分别存储秒和纳秒信息
4.  【强制】不要在程序中写死一年为 365 天，避免在公历闰年时出现日期转换错误或程序逻辑
错误

#### 四、集合处理
1. 【强制】关于 hashCode 和 equals 的处理，遵循如下规则：
1） 只要覆写 equals，就必须覆写 hashCode。
2） 因为 Set 存储的是不重复的对象，依据 hashCode 和 equals 进行判断，所以 Set 存储的对象必须覆写这两种方法
3） 如果自定义对象作为 Map 的键，那么必须覆写 hashCode 和 equals
2.  【强制】判断所有集合内部的元素是否为空，使用 isEmpty()方法，而不是 size()==0 的方式
3.  【强制】在使用 java.util.stream.Collectors 类的 toMap()方法转为 Map 集合时，一定要使用含有参数类型为 BinaryOperator，参数名为 mergeFunction 的方法，否则当出现相同 key值时会抛出 IllegalStateException 异常。
说明：JDK1.8后的版本
4.  【强制】在使用 java.util.stream.Collectors 类的 toMap()方法转为 Map 集合时，一定要注意当 value 为 null 时会抛 NPE 异常
5.   【强制】ArrayList 的 subList 结果不可强转成 ArrayList，否则会抛出 ClassCastException 异常：java.util.RandomAccessSubList cannot be cast to java.util.ArrayList。说明：subList()返回的是 ArrayList 的内部类 SubList，并不是 ArrayList 本身，而是 ArrayList 的一个视图，对于 SubList 的所有操作最终会反映到原列表上
6.  【强制】使用 Map 的方法 keySet()/values()/entrySet()返回集合对象时，不可以对其进行添加元素操作，否则会抛出 UnsupportedOperationException 异常
7.   【强制】Collections 类返回的对象，如：emptyList()/singletonList()等都是 immutable list，不可对其进行添加或者删除元素的操作
8. 【强制】在 subList 场景中，高度注意对父集合元素的增加或删除，均会导致子列表的遍历、
增加、删除产生 ConcurrentModificationException 异常
9. 【强制】使用集合转数组的方法，必须使用集合的 toArray(T[] array)，传入的是类型完全一致、长度为 0 的空数组
反例：直接使用 toArray 无参方法存在问题，此方法返回值只能是 Object[]类，若强转其它类型数组将出现ClassCastException 错误
10.  【强制】在使用 Collection 接口任何实现类的 addAll()方法时，都要对输入的集合参数进行NPE 判断
说明：在 ArrayList#addAll 方法的第一行代码即 Object[] a = c.toArray(); 其中 c 为输入集合参数，如果为 null，则直接抛出异常
11.  【强制】使用工具类 Arrays.asList()把数组转换成集合时，不能使用其修改集合相关的方法， 它的 add/remove/clear 方法会抛出 UnsupportedOperationException 异常
12.   【强制】泛型通配符<? extends T>来接收返回的数据，此写法的泛型集合不能使用 add 方法， 而<? super T>不能使用 get 方法，两者在接口调用赋值的场景中容易出错
说明：扩展说一下 PECS(Producer Extends Consumer Super)原则：第一、频繁往外读取内容的，适合用<? extends T>。第二、经常往里插入的，适合用<? super T>
13. 【强制】在无泛型限制定义的集合赋值给泛型限制的集合时，在使用集合元素时，需要进行instanceof 判断，避免抛出 ClassCastException 异常
14. 【强制】不要在 foreach 循环里进行元素的 remove/add 操作。remove 元素请使用 Iterator方式，如果并发操作，需要对 Iterator 对象加锁

#### 五、并发处理
1. 【强制】获取单例对象需要保证线程安全，其中的方法也要保证线程安全
2. 【强制】创建线程或线程池时请指定有意义的线程名称，方便出错时回溯
3. 【强制】线程资源必须通过线程池提供，不允许在应用中自行显式创建线程
说明：线程池的好处是减少在创建和销毁线程上所消耗的时间以及系统资源的开销，解决资源不足的问题。如果不使用线程池，有可能造成系统创建大量同类线程而导致消耗完内存或者“过度切换”的问题
4.  【强制】SimpleDateFormat 是线程不安全的类，一般不要定义为 static 变量，如果定义为static必须加锁，或者使用 DateUtils 工具类
正例：注意线程安全，使用 DateUtils。亦推荐如下处理
```
private static final ThreadLocal<DateFormat> df = new ThreadLocal<DateFormat>() {
 @Override
 protected DateFormat initialValue() {
 	return new SimpleDateFormat("yyyy-MM-dd");
 }
};
```
说明：如果是 JDK8 的应用，可以使用 Instant 代替 Date，LocalDateTime 代替 Calendar， DateTimeFormatter 代替 SimpleDateFormat，官方给出的解释：simple beautiful strong immutable thread-safe
5. 【强制】必须回收自定义的 ThreadLocal 变量，尤其在线程池场景下，线程经常会被复用， 如果不清理自定义的 ThreadLocal 变量，可能会影响后续业务逻辑和造成内存泄露等问题。 尽量在代理中使用 try-finally 块进行回收
正例：
```
objectThreadLocal.set(userInfo);
try {
 // ...
} finally {
 objectThreadLocal.remove();
}
```
6. 【强制】高并发时，同步调用应该去考量锁的性能损耗。能用无锁数据结构，就不要用锁；能 锁区块，就不要锁整个方法体；能用对象锁，就不要用类锁
说明：尽可能使加锁的代码块工作量尽可能的小，避免在锁代码块中调用 RPC 方法
7. 【强制】对多个资源、数据库表、对象同时加锁时，需要保持一致的加锁顺序，否则可能会造成死锁
说明：线程一需要对表 A、B、C 依次全部加锁后才可以进行更新操作，那么线程二的加锁顺序也必须是 A、B、C，否则可能出现死锁
8. 【强制】在使用阻塞等待获取锁的方式中，必须在 try 代码块之外，并且在加锁方法与 try 代 码块之间没有任何可能抛出异常的方法调用，避免加锁成功后，在 finally 中无法解锁。
说明一：如果在 lock 方法与 try 代码块之间的方法调用抛出异常，那么无法解锁，造成其它线程无法成功获取锁。
说明二：如果 lock 方法在 try 代码块之内，可能由于其它方法抛出异常，导致在 finally 代码块中，unlock对未加锁的对象解锁，它会调用 AQS 的 tryRelease 方法（取决于具体实现类），抛出
IllegalMonitorStateException 异常。
说明三：在 Lock 对象的 lock 方法实现中可能抛出 unchecked 异常，产生的后果与说明二相同
正例：
```
Lock lock = new XxxLock();
// ...
lock.lock();
try {
 doSomething();
 doOthers();
} finally {
 lock.unlock();
}
```
反例：
```
Lock lock = new XxxLock();
try {
 // 如果此处抛出异常，则直接执行 finally 代码块
 doSomething();
 // 无论加锁是否成功，finally 代码块都会执行
 lock.lock();
 doOthers();
} finally {
 lock.unlock();
}
```
9.  【强制】在使用尝试机制来获取锁的方式中，进入业务代码块之前，必须先判断当前线程是否持有锁。锁的释放规则与锁的阻塞等待方式相同
说明：Lock 对象的 unlock 方法在执行时，它会调用 AQS 的 tryRelease 方法（取决于具体实现类），如果 当前线程不持有锁，则抛出 IllegalMonitorStateException 异常
```java
Lock lock = new XxxLock();
// ...
boolean isLocked = lock.tryLock();
if (isLocked) {
 try {
 doSomething();
 doOthers();
 } finally {
 lock.unlock();
 }
}
```
10. 【强制】并发修改同一记录时，避免更新丢失，需要加锁。要么在应用层加锁，要么在缓存加 锁，要么在数据库层使用乐观锁，使用 version 作为更新依据。
说明：如果每次访问冲突概率小于 20%，推荐使用乐观锁，否则使用悲观锁。乐观锁的重试次数不得小于3 次。
11. 【推荐】使用 CountDownLatch 进行异步转同步操作，每个线程退出前必须调用 countDown 方 法，线程执行代码注意 catch 异常，确保 countDown 方法被执行到，避免主线程无法执行至 await 方法，直到超时才返回结果。
说明：注意，子线程抛出异常堆栈，不能在主线程 try-catch 到
12.  【推荐】避免 Random 实例被多线程使用，虽然共享该实例是线程安全的，但会因竞争同一 seed导致的性能下降
说明：Random 实例包括 java.util.Random 的实例或者 Math.random()的方式。
正例：在 JDK7 之后，可以直接使用 API ThreadLocalRandom，而在 JDK7 之前，需要编码保证每个线程持有一个单独的 Random 实例
13. 【参考】ThreadLocal 对象使用 static 修饰，ThreadLocal 无法解决共享对象的更新问题

#### 六、前、后端公约
1. 【强制】前后端交互的 API，需要明确协议、域名、路径、请求方法、请求内容、状态码、响应体
2.  【强制】前后端数据**列表**相关的接口返回，如果为空，则返回空数组[]或空集合{}
说明：此条约定有利于数据层面上的协作更加高效，减少前端很多琐碎的 null 判断
3.  【强制】在前后端交互的 JSON 格式数据中，所有的 key 必须为小写字母开始的lowerCamelCase 风格，符合英文表达习惯，且表意完整
4.   【强制】对于需要使用超大整数的场景，服务端一律使用 String 字符串类型返回，禁止使用Long 类型
5.   【强制】HTTP 请求通过 URL 传递参数时，不能超过 2048 字节
6.   【强制】HTTP 请求通过 body 传递内容时，必须控制长度，超出最大长度后，后端解析会出错
说明：nginx 默认限制是 1MB，tomcat 默认限制为 2MB，当确实有业务需要传较大内容时，可以通过调大服务器端的限制。
Nginx调整参数：client_max_body_size
tomcat调整参数：server.xml中的节点添加maxPostSize
7.   【强制】在翻页场景中，用户输入参数的小于 1，则前端返回第一页参数给后端；后端发现用户输入的参数大于总页数，直接返回最后一页
8.   【强制】服务器内部重定向必须使用 forward；外部重定向地址必须使用 URL 统一代理模块生成，否则会因线上采用 HTTPS 协议而导致浏览器提示“不安全”，并且还会带来 URL 维护不一致的问题
9.   【推荐】前后端的时间格式统一为"yyyy-MM-dd HH:mm:ss"，统一为 GMT
10. 【参考】在接口路径中不要加入版本号，版本控制在 HTTP 头信息中体现，有利于向前兼容

#### 七、其他
1. 【强制】finally 块必须对资源对象、流对象进行关闭，有异常也要做 try-catch
说明：如果 JDK7 及以上，可以使用 try-with-resources 方式
2. 【强制】不要在 finally 块中使用 return
说明：try 块中的 return 语句执行成功后，并不马上返回，而是继续执行 finally 块中的语句，如果此处存在 return 语句，则在此直接返回，无情丢弃掉 try 块中的返回点
3. 【强制】应用中不可直接使用日志系统（Log4j、Logback）中的 API，而应依赖使用日志框架（SLF4J、JCL--Jakarta Commons Logging）中的 API，使用门面模式的日志框架，有利于维护和各个类的日志处理方式统一
4. 【强制】在日志输出时，字符串变量之间的拼接使用占位符的方式
说明：因为 String 字符串的拼接会使用 StringBuilder 的 append()方式，有一定的性能损耗。使用占位符仅是替换动作，可以有效提升性能。
正例：logger.debug("Processing trade with id: {} and symbol: {}", id, symbol);
5. 【强制】生产环境禁止直接使用 System.out 或 System.err 输出日志或使用e.printStackTrace()打印异常堆栈