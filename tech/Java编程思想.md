# Java编程思想

`第4版`

***

## 目录

[TOC]

***

## 面向对象的设计思想

    1.设计类的范围: 每个类的对象可以很好的完成一项任务, 但并不试图做更多
    2.隐藏具体实现: 通过接口或protected、private关键字, 隐藏类的属性和具体实现, 以便可以安全的进行重构
    3.复用方式的选择: 优先考虑更加简单灵活的组合(has-a), 当新类与现有类具有很多相似的功能时, 应该考虑继承(is-a)
    4.基本成员默认值: 当变量作为类的成员变量时, Java会确保给定默认值, 但变量作为局部变量时, 不会被自动初始化

## 类与对象

    1.package关键字: 定义类库(或命名空间), 可通过import引入类库
    2.import static: 可以导入静态方法
    3.方法重载: 方法名相同, 参数类型列表不同
        方法重写: 导出类重写基类的方法
    4.由于Java的单根继承, 将垃圾回收相关的工作都放在基类(Object)中实现
    5.初始化顺序: 基类静态成员/块->子类静态成员/块->基类非静态成员/块->基类构造器->子类非静态成员/块->子类构造器
        静态初始化块执行时机: 首次生成类的对象时或首次访问类的静态成员(构造方法也属于静态方法)时
    6.final关键字: 强调不能更改
        修饰参数: 禁止修改
        修饰方法: 禁止重写
        修饰类: 禁止继承
    7.static关键字: 静态的, 强调只有一份, static成员或方法属于类
    8.protected关键字: 对于导出类是可访问的, 同时具有包访问权限
    9.动态绑定: 在运行时根据对象的类型进行绑定, 也叫后期绑定或运行时绑定, 是Java中多态方法调用的实现方式
        Java中除了static方法(属于类)和final方法(包括private方法), 其他方法都是后期绑定, Java中的域不具有多态性
    10.内部类: 在另一个类(外部类)的定义内部, 内部类可以访问外围对象的所有成员, 在其他地方创建内部类时, 必须使用外部类的对象来创建
    11.嵌套类: static内部类

## 类型信息

    1.Class对象: 每个类都有一个Class对象, Java使用Class对象来执行RTTI
    2.类加载器: 检查某个类的Class对象是否已经加载, 若尚未加载, 默认的类加载器会根据类名查找'.class'文件. 所有的类在对其第一次使用时, 动态加载到JVM中
    3.类加载步骤:
        1).加载: 查找字节码, 创建Class对象
        2).连接: 验证类中的字节码, 为静态域分配存储空间
        3).初始化: 执行静态初始化器或静态初始化块
    4.内省: 在运行时检查一个对象的类型或者属性, 如:instanceof关键字
        反射: 在运行时检查或者修改一个对象信息
    5.Java反射机制之Class: 反射的入口
        1).获取Class对象的方式:
            a.Object.getClass(): 调用已有对象的getClass()方法
            b.Object.class: 类名后加.class, 也可用于基本类型和数组, 如: long.class, int[][].class
            c.Class.forName(): 通过类的完整路径得到Class对象, 只能用于引用类型
            d.静态属性Type: 用于基本类型和void的包装类, 如: Void.Type, Integer.Type
            e.通过已有Class对象: Class.getSuperclass(), Class.getClasses(), Class.getDeclaredClasses()等
            f.getDeclaringClass(): Class, Field, Method, Constructor都有该方法, 返回对象所在的类
        2).Class修饰符: Modifier
            Modifier集合: public, protected, private, abstract, static, final, native, synchronized, ...
            用法:
                int modifiers = Class.getModifiers(): 获得调用类的修饰符的二进制值
                Modifier.toString(modifiers); // 将二进制值转换为字符串, 如: "public final"
                Modifier.isFinal(modifiers); // 判断是否存在final修饰符
        3).Class的成员: java.lang.reflect.Member接口
            三个实现类: Field(成员变量), Method(成员方法), Constructor(构造函数)
            获取方法: Class.getDeclaredXxx(String name), getXxx(String name), getDeclaredXxxs(), getXxxs()
    6.Java反射机制之Field
        1).Field.getType(): 返回变量类型
            Field.getGenericType(): 如果当前属性有签名属性类型就返回, 否则返回Field.getType()
        2).常见异常:
            a.NoSuchFieldException: 变量不存在或调用Class.getField()访问private的变量时
                解决方案: 使用Class.getDeclaredField()
            b.IllegalAccessException: 获取或修改private的变量时
                解决方案: Field.setAccessible(true)
            c.IllegalArgumentException: 调用Field.getInt(), Field.setInt()等操作基本类型的方法, 而原变量为包装类时
                如: field.getLong(obj), field.setInt(obj, 10)
                原因: 使用反射获取或者修改一个变量的值时, 编译器不会进行自动装/拆箱
                解决方案: 赋值为包装类对象, 如: field.set(obj, new Integer(10))
    7.Java反射机制之Method
        Method.getName(): 返回方法名
        Method.getModifiers(): 返回方法修饰符
        Method.getReturnType(): 方法返回类型
        Method.isAnnotationPresent(Class<? extends Annotation> clazz): 是否存在注解
        Method.isBridge(): 是否是桥接方法, 泛型机制, 编译器有时需要自动生成桥接方法
        Method.isSynthetic(): 是否是合成方法, java允许外部类访问内部类私有成员, 编译器自动生成
        Method.isDefault(): 是否是默认方法, jdk8中接口的default方法
        ...
        Method.invoke(Object obj, Object... args): 执行方法
    8.Java反射机制之Constructor
        Constructor.newInstance(Object... args): 实例化对象
    9.动态代理: Proxy.newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)
        原理: JDK生成新的class, 继承Proxy类, 并实现需要代理的接口interfaces, 实现方法调用h.invoke()
        Java不支持继承多个class, 且生成的代理类已经继承了Proxy类, 因此JDK动态代理只能代理接口
        CGLib动态代理: 支持接口或类的动态代理(创建子类), 代理方法不能为final方法(无法重写)
        
## 泛型

    1.泛型机制的实现:
        C++泛型: 实例化时为每一个泛型类或泛型方法产生一份新代码(代码膨胀)
        Java泛型: 生成唯一一份目标代码, 使用时执行类型检查和类型转换(伪泛型, 向前兼容)
    2.类型检查的依据: 引用类型
        List<String> list = new ArrayList<>(), list.add()只能接受String类型
        List list = new ArrayList<String>(), list.add()可以接受任何Object对象
        类型检查是有"List<String>"或"List"来决定的
        特殊情况: 匿名方式创建对象时, 如: (new ArrayList<String>()).add()也只能接受String类型
    3.泛型中的桥方法: 保证Java的多态, 示例:
        1).类: Holder<T>
            存在方法:
                public void setObj(T t) {...}
                public T getObj() {...}
            编译时由于类型擦除, 生成的方法为:
                public void setObj(Object t) {...} // 方法1
                public Object getObj() {...} // 方法2
        2).类: StringHolder extends Holder<String>
            方法为:
                public void setObj(String t) {...} // 方法3
                public String getObj() {...} // 方法4
        3).分析:
            方法3和方法1不是重写, 而是重载, 因此java虚拟机会生成一个桥方法, 来保持多态
            方法4和方法2属于协变类型, 但java虚拟机也会生成一个桥方法, 桥方法与原方法仅返回类型不同, 但虚拟机可以正确处理这一情况
        4).结论: 在子类StringHolder中, 实际生成了4个方法
            public void setObj(String t) {...}
            public void setObj(Object t) {setObj((String) t)} // 桥方法
            public String getObj() {...}
            public Object getObj() {...} // 桥方法, 在实际编码中若两个方法仅返回类型不同会发生冲突
    4.List、List<Object>、List<?>的区别:
        List和List<Object>表示持有Object类型的原生List,
        List<?>表示持有某种特定类型的非原生List, 只是不知道具体类型是什么, 因此只能get(), 不能add()/set()
        对于Map, 可以限定其中一个泛型, 如: Map<String, ?>
    5.自限定类型: class A extends Page<A> {}, 基类用导出类替代其参数
    
## 注解

    1.三种内置标准注解:
        @Override: 覆盖超类方法
        @Deprecated: 已废弃
        @SuppressWarning: 关闭不当的编译器警告信息
    2.元注解(4种): 用来定义注解
        1).@Target: 注解用于什么地方
            ElementType参数:
                TYPE: 类、接口(包括注解)或enum声明
                FIELD: 域声明(包括enum实例)
                METHOD: 方法声明
                PARAMETER: 参数声明
                CONSTRUCTOR: 构造器声明
                LOCAL_VARIABLE: 局部变量声明
                PACKAGE: 包声明
        2).@Retention: 表示在什么级别保存注解信息
            RetentionPolicy参数:
                SOURCE: 注解将被编译器丢弃
                CLASS: 注解在class文件中可用, 但会被VM丢弃
                RUNTIME: VM将在运行期保留注解, 可以通过反射机制读取注解信息
        3).@Documented: 将注解包含在Javadoc中
        4).@Inherited: 允许子类继承父类的注解
    3.注解元素类型: 基本类型(int, float, boolean等), String, Class, enum, Annotation, 以上类型的数组
    4.标记注解: 没有元素的注解
    4.注解不支持继承
    
## 并发

    1.线程: Thread, 切分CPU时间片
        Thread.yield(): 对线程调度的一种建议, 建议此时可以切换任务
        Thread.sleep(): 休眠, 若此时[中断标志位]为true, 则抛出InterruptedException
            注1: 任务处于可中断阻塞(调用Thread.sleep()、Thread.join()或Object.wait()), 且[中断标志位]为true, 则抛出InterruptedException, 并清除[中断标志位]状态
            注2: I/O操作或正在获取锁的线程不能被中断, 如等待synchronized锁时的阻塞不是可中断阻塞, 但在synchronized代码块中调用Thread.sleep()、Thread.join()或Object.wait()造成的阻塞可以被中断
        Thread.interrupt(): 设置[中断标志位]为true
        Thread.isInterrupted(flag): 获取中断状态, 若flag为true则清除[中断标志位]状态
        Thread.join(): 调用目标线程的join()方法, 当前线程被挂起(阻塞), 直至目标线程结束才恢复
        Thread.setPriority(priority): 设置线程的优先级, 线程调度具有不确定性, 优先级高的线程有较大的几率先执行
        Thread.setDaemon(true): 将线程设置为后台线程, 且后台线程中创建的线程将被自动设置为后台线程
            注: 当最后一个非后台线程终止时, 后台线程会突然"终止", 即使是finally子句也得不到执行
    2.任务: Runnable, 创建线程代价高昂, 因此将任务(Runnable)与线程(Thread)分离
    3.异常处理器: Thread.UncaughtExceptionHandler接口, Thread.setDefaultUncaughtExceptionHandler()设置异常处理器
        注1: 当线程因未捕获的异常而临近死亡时将调用异常处理器
        注2: run()方法中逃逸的异常会向外传播到控制台, 因此需要采用特殊步骤捕获异常
    4.执行器: Executor, 如:
        ExecutorService threadPool = Executor.newCachedThreadPool();
        ExecutorService执行任务的方法:
            1).execute(Runnable r), 执行任务
            2).submit(Callback<T> C), 返回Future<T>, Future<T>.get()方法可以接收Callback<T>.call()方法的返回值, 若调用Future<T>.get()时任务未返回, 将阻塞直至结果准备就绪
    5.volatile关键字: 确保可见性, volatile域会被立即写入到主存中(读取操作发生在主存中)
        注1: java虚拟机机制, 全局变量会在方法执行的栈中保存副本, 在不同步的情况下, 对变量的修改要等到方法执行完毕才会更新到全局
        注2: synchronized关键字包住的代码块是同步的, 会导致向主存中刷新
        原子性: 保证任务作出的修改是不被中断的, 但不一定是可见的. 对于i++运算不是原子性的, 分为get、add和put
    6.synchronized关键字: 内建锁机制, 锁的目标: 对象
        1).synchronized方法: 锁this对象, static synchronized方法: 锁Class对象
            因此: 所有synchronized方法共享同一个锁(this), 所有static synchronized方法共享同一个锁(Class)
        2).synchronized锁是可重入锁, 可以在一个synchronized方法中调用其他synchronized方法, 计数加1
    7.线程协作: wait()和notify()/notifyAll()
        注: 调用wait()和notify()/notifyAll()方法必须在同步控制代码块之内, 因为调用他们之前必须先拥有锁
        1).Object.wait(): 释放锁(Object的锁), Thread.sleep()和Thread.yield()方法不释放锁
            释放锁的时机: 调用Object.wait()或执行完synchronized代码块
    8.Lock: 显示锁机制, 手动调用Lock.lock()和Lock.unlock()实现加锁和释放锁, 更加灵活
        Condition: 可通过Lock.newCondition()获取, Condition.await()和signal()/signalAll()类似于synchronized代码块中的Object.wait()和notify()/notifyAll()
        
## Java8新特性

    1.lambda表达式: () -> {}
        目标类型: 函数式接口(只包含一个抽象方法的接口)
        若只有一个形参, 可以省略圆括号; 若代码块只有一个语句, 可以去掉花括号, 并返回语句的值
        lambda表达式只能用当前类型接收, 赋值给父类时需要先进行转换, 如:
            Object obj = (Runnable) () -> System.out.println("runnable接口");
        若只有一个参数和一句语句, 可以简写为"Object::method"形式, 如:
            list.forEach(str -> System.out.println(str)); 等价于 list.forEach(System.out::println);
    2.Stream API: 对集合(Collection)对象功能的增强, 类似一个高级版本的Iterable
        1).作用: 对集合对象进行各种非常便利、高效的聚合操作(aggregate operation), 或者大批量数据操作(bulk data operation)
        2).效率: 同时提供串行和并行两种模式进行汇聚操作, 并发模式能够充分利用多核处理器的优势, 使用"fork/join"并行方式来拆分任务和加速处理过程
        3).生成Stream Source方式
            Collection.stream()
            Collection.parallelStream()
            Arrays.stream(T array)
            Stream.of(T t) or Stream.of(T... values)
            java.io.BufferedReader.lines()
            java.util.stream.IntStream.range()
            Pattern.splitAsStream()
            ...
        4).三种包装类型Stream: IntStream, LongStream, DoubleStream
            效率高于Stream<Integer>, Stream<Long>, Stream<Double>
        5).Stream操作类型:
            a.intermediate: 一个流可以后面跟随零个或多个intermediate操作. 其目的主要是打开流, 做出某种程度的数据映射/过滤, 然后返回一个新的流, 交给下一个操作使用, 这类操作都是惰性化的(lazy), 仅仅调用到这类方法, 并没有真正开始流的遍历
            b.terminal: 一个流只能有一个terminal操作, 当这个操作执行后, 流就被使用"光"了, 无法再被操作, 所以这必定是流的最后一个操作, terminal操作的执行, 才会真正开始流的遍历, 并且会生成一个结果, 或者一个side-effect
            c.short-circuiting: 当操作一个无限大的Stream, 而又希望在有限时间内完成操作时, 则在管道内拥有一个short-circuiting操作是必要非充分条件
        6).Stream操作方法:
            a.intermediate类型: map(mapToInt, flatMap等), filter, distinct, sorted, peek, limit, skip, parallel, sequential, unordered
            b.terminal类型: forEach, forEachOrdered, toArray, reduce, collect, min, max, count, anyMatch, allMatch, noneMatch, findFirst, findAny, iterator
            c.short-circuiting类型: anyMatch, allMatch, noneMatch, findFirst, findAny, limit
    3.default关键字: 在接口中进行扩展, 打破Java之前版本对接口的语法限制
        default方法也叫虚拟扩展方法(virtual extension methods), 扩展接口可以不用重写default方法, 但出现多继承default方法冲突时, 实现类必须重写default方法
        
