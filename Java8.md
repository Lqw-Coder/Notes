## Java8

### 1 内置的四大核心函数式接口

* Consumer<T> ：消费型接口
* Supplier<T>: 供给型接口
* Function<T,R> :函数型接口
* Predicate<T>:断言型接口

### 2 方法引用（Lambda体已有方法实现）

```java
     @Test
    public void testMethod(){
        //java8方法引用
        //对象::实例方法名
        Consumer<String> c = System.out::println;
        Consumer<String> c1 = (x) -> System.out.println(x);
        c.accept("c");
        c1.accept("c1");
        //类::静态方法名
        Comparator<Integer> comparator = Integer::compare;
        //类::实例方法名
        BiFunction<String,String, Boolean> biFunction = String::equals;
        System.out.println(biFunction.apply("c1","c1"));//注意
        //构造方法调用
        Supplier<Person> p = Person::new;
        Function<String,Person> p2 = Person::new;
        p2.apply("p2");
        Function<Integer,int[]> p3 = int[]::new;
        int[] a = p3.apply(20);
        System.out.println(a.length);
    }
```

### 3 StreamAPI

* Stream的创建（会将数据源转换成Stream流，源数据不会受到影响）

```java
    @Test
    public void testStream(){
        //Stream的创建
        //1 可以通过Collection系列集合提供的Stream()或 parallelStream()
        List<String> list = new ArrayList<>();
        Stream<String> stream1 = list.stream();

        //2 通过Arrays中的静态方法stream()获取数组流
        Person[]persons = new Person[10];
        Stream<Person> stream = Arrays.stream(persons);

        //3 通过Stream类的静态方法of()
        Stream<String> stream2 = Stream.of("你好","5555");

        //4 创建无限流
        //迭代
        Stream<Integer> stream3 = Stream.iterate(0,(x)-> x+2);
        stream3.limit(10).forEach(System.out::println);
        //生成
        Stream.generate(()->Math.random())
                .limit(10)
                .forEach(System.out::println);
    }
```

* 中间操作（惰性加载，即无终止操作不会执行中间操作）

```java
    /**
     * 筛选与切片
     *      filter——接收 Lambda ， 从流中排除某些元素。
     *      limit——截断流，使其元素不超过给定数量。
     *      skip(n) —— 跳过元素，返回一个扔掉了前 n 个元素的流。若流中元素不足 n 个，则返回一个空流。
     *      与 limit(n) 互补
     *      distinct——筛选，通过流所生成元素的 hashCode() 和 equals() 去除重复元素
     * 映射
     *      map——接收 Lambda ， 将元素转换成其他形式或提取信息。接收一个函数作为参数，该函数会
     *      被应用到每个元素上，并将其映射成一个新的元素。
     *      flatMap——接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流
     *      类比 addAll(Collection c) 跟 add(Collection c)
     * 排序
     *      sorted() -- 自然排序，即类自身实现的方法
     *      sorted(Comparator com) -- 定制排序
     * */
```

```java
    @Test
    public void test1(){
        //map 与 flatMap的区别
        List<String> strList = Arrays.asList("aaa", "bbb", "ccc", "ddd", "eee");
        Stream<Stream<Character>> stream3 = strList.stream()
                .map(LambdaDemo::filterCharacter);
        Stream<Character> stream4 = strList.stream()
                .flatMap(LambdaDemo::filterCharacter);
        stream3.forEach(System.out::println);
        stream4.forEach(System.out::println);
    }
    public static Stream<Character> filterCharacter(String str){
        List<Character> list = new ArrayList<>();
        for (Character ch : str.toCharArray()) {
            list.add(ch);
        }
        return list.stream();
    }
```

* 终止操作（流在进行终止操作后不能再次使用）

```java
    /**
     *      allMatch——检查是否匹配所有元素
     * 		anyMatch——检查是否至少匹配一个元素
     * 		noneMatch——检查是否没有匹配的元素
     * 		findFirst——返回第一个元素
     * 		findAny——返回当前流中的任意元素
     * 		count——返回流中元素的总个数
     * 		max——返回流中最大值
     * 		min——返回流中最小值
     * 	    归约——可以将流中元素反复结合起来，得到一个值。
     * 	    reduce(T identity, BinaryOperator) / reduce(BinaryOperator) 
     * 	    BinaryOperator 二元运算
     * 	    收集（分组以及分区）collect(),可以通过Collectors来获取收集器
     * 	    将流转换为其他形式。接收一个 Collector接口的实现，用于给Stream中元素做汇总的方法
     * */
```

```java
   @Test
    public void test2(){
        //max操作
        List<Person> people = new ArrayList<>();
        people.add(new Person(20));
        people.add(new Person(30));
        people.add(new Person(25));
        Optional<Integer> option = people.stream().map(Person::getAge).max((x,y)->x-y);
        System.out.println(option.get());
        //收集操作
        Set<Person> set = people.stream().collect(Collectors.toSet());//转换为Set集合
        List<Person> list = people.stream().collect(Collectors.toList());//转换为list集合
        HashSet<Person> hashSet = 			       people.stream().collect(Collectors.toCollection(HashSet::new));//转换为想要的集合
    }
    @Test
    public void test3(){
        //归约操作
        List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9,10);
        Integer sum = list.stream()
                .reduce(0, (x, y) -> x + y);
        System.out.println(sum);
        Optional<Integer> optionalInteger = list.stream().reduce(Integer::sum);
        System.out.println(optionalInteger.get());
    }
```

```java
	List<Employee> emps = Arrays.asList(
			new Employee(102, "李四", 79, 6666.66, Status.BUSY),
			new Employee(101, "张三", 18, 9999.99, Status.FREE),
			new Employee(103, "王五", 28, 3333.33, Status.VOCATION),
			new Employee(104, "赵六", 8, 7777.77, Status.BUSY),
			new Employee(104, "赵六", 8, 7777.77, Status.FREE),
			new Employee(104, "赵六", 8, 7777.77, Status.FREE),
			new Employee(105, "田七", 38, 5555.55, Status.BUSY)
	);
	//分组
	@Test
	public void test5(){
		Map<Status, List<Employee>> map = emps.stream()
			.collect(Collectors.groupingBy(Employee::getStatus));
		
		System.out.println(map);
	}
	//分区
	@Test
	public void test7(){
		Map<Boolean, List<Employee>> map = emps.stream()
			.collect(Collectors.partitioningBy((e) -> e.getSalary() >= 5000));
		
		System.out.println(map);
	}
```

* 并行流

#### ForkJoin框架与传统线程池的区别

ForkJoin采用了“工作窃取”模式，即若有一个线程无任务执行，则会至其他线程中取出一个任务进行执行，这样就不会造成部分线程无任务执行，而其他线程则处于执行状态

```java
public class ForkJoinCalculate extends RecursiveTask<Long> {
    private static final long serialVersionUID = 13475679780L;
    private long start;
    private long end;
    private static final long THRESHOLD = 10000L;
    public ForkJoinCalculate(long start,long end){
        this.start = start;
        this.end = end;
    }
    @Override
    protected Long compute() {
        long length = end - start;

        if(length <= THRESHOLD){
            long sum = 0;

            for (long i = start; i <= end; i++) {
                sum += i;
            }

            return sum;
        }else{
            long middle = (start + end) / 2;

            ForkJoinCalculate left = new ForkJoinCalculate(start, middle);
            left.fork(); //拆分，并将该子任务压入线程队列
            ForkJoinCalculate right = new ForkJoinCalculate(middle+1, end);
            right.fork();

            return left.join() + right.join();
        }
    }
}
```

```java
    @Test
    public void test(){
        long start = System.currentTimeMillis();
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinTask<Long> task = new ForkJoinCalculate(0L,10000000000L);
        long sum = forkJoinPool.invoke(task);
        long end = System.currentTimeMillis();
        System.out.println((end - start));
    }
    @Test
    public void test3(){
        long start = System.currentTimeMillis();
        //Stream API 可以声明性地通过 parallel() 与sequential() 在并行流与顺序流之间进行切换
        Long sum = LongStream.rangeClosed(0L, 10000000000L)
                .parallel()//转换为并行流
                .sum();
        System.out.println(sum);
        long end = System.currentTimeMillis();
        System.out.println("耗费的时间为: " + (end - start)); //2061-2053-2086-18926
    }
```

### 4 Optional

Optional着重为解决java的NPE问题。善用Optional可以使我们代码中很多繁琐、丑陋的设计变得十分优雅，特别是嵌套判断

```java
    /**
     * 一、Optional 容器类：用于尽量避免空指针异常
     * Optional.of(T t) : 创建一个 Optional 实例
     * Optional.empty() : 创建一个空的 Optional 实例
     * Optional.ofNullable(T t):若 t 不为 null,创建 Optional 实例,否则创建空实例
     * isPresent() : 判断是否包含值
     * orElse(T t) :  如果调用对象包含值，返回该值，否则返回t
     * orElseGet(Supplier s) :如果调用对象包含值，返回该值，否则返回 s 获取的值
     * map(Function f): 如果有值对其处理，并返回处理后的Optional，否则返回 Optional.empty()
     * flatMap(Function mapper):与 map 类似，要求返回值必须是Optional
     */
```



```java
public static String getChampionName(Competition comp) throws IllegalArgumentException {
    if (comp != null) {
        CompResult result = comp.getResult();
        if (result != null) {
            User champion = result.getChampion();
            if (champion != null) {
                return champion.getName();
            }
        }
    }
    throw new IllegalArgumentException("The value of param comp isn't available.");
}

让我们看看经过Optional加持过后，这些代码会变成什么样子。
public static String getChampionName(Competition comp) throws IllegalArgumentException {
    return Optional.ofNullable(comp)
            .map(c->c.getResult())
            .map(r->r.getChampion())
            .map(u->u.getName())
            .orElseThrow(()->new IllegalArgumentException("The value of param comp isn't available."));
}
```



