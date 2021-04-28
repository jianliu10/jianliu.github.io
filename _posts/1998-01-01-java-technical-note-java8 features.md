---
layout: post
title:  "Java 8 features"
date:   2018-10-01 13:15:42 -0500
categories: tech-java-spring
---

https://mp.weixin.qq.com/s/_YVS4MNn6jZc_kfEX4vcZA


Java8新特性中最为重要的便是Lambda表达式和Stream API了，先来了解一下Lambda表达式吧。

## Lambda表达式

Lambda表达式是一个匿名函数，我们可以将Lambda表达式理解为一段可以作为参数传递的代码，通过Lambda表达式，我们可以将Java程序变得更加简洁和灵活。

来看一段程序：

	@Test
	public void test() {
		Comparator<Integer> comparator = new Comparator<Integer>() {
			@Override
			public int compare(Integer o1, Integer o2) {
				return o2 - o1;
			}
		};
		Set<Integer> set = new TreeSet<>(comparator);
		set.add(3);
		set.add(2);
		set.add(1);
		System.out.println(set);
	}

在JDK8以前，若是想实现对TreeSet的自定义排序，我们可以创建匿名的比较器类将其传入TreeSet的构造方法，而JDK8之后，使用Lambda表达式可以简化匿名内部类的代码：

	@Test
	public void test() {
		Comparator<Integer> comparator = (o1, o2) -> o2 - o1;
		Set<Integer> set = new TreeSet<>(comparator);
		set.add(3);
		set.add(2);
		set.add(1);
		System.out.println(set);
	}

原本匿名内部类的创建现在直接被简化成了一行代码：

Comparator<Integer> comparator = (o1, o2) -> o2 - o1;
下面就来看看Lambda该如何使用，也就是学习一下它的语法。


### Lambda表达式语法

在Java8中新引入了一个操作符：-> ，它被称为箭头操作符或Lambda操作符，箭头操作符将Lambda表达式拆分成了左侧和右侧的两部分，其中左侧为Lambda表达式的 参数列表 ，右侧为Lambda表达式的功能代码，也叫 Lambda体 。对应一个具体的Lambda表达式：

Comparator<Integer> comparator = (o1, o2) -> o2 - o1;
此时箭头操作符的左侧就是Comparator接口中compare方法的参数列表，右侧即为该方法所需要实现的功能：

	@FunctionalInterface
	public interface Comparator<T> {
		int compare(T o1, T o2);
		......
	}

基于接口中方法声明的不同，Lambda表达式的编写方式也会随之发生相应的变化，大体分为以下几类：

无参无返回值：Runnable runnable = () -> System.out.println("Hello Lambda!");
有一个参数但无返回值：Consumer consumer = (s) -> System.out.println(s);
若是方法只有一个参数，则可以省略小括号，所以可以简写为：Consumer consumer = s -> System.out.println(s);
有多个参数有返回值，且Lambda体中有多条语句：

	Comparator<Integer> comparator = (o1, o2) -> {
		System.out.println("从大到小排列");
		return o2 - o1;
	};

若是Lambda体中有多条语句，则Lambda体必须用大括号包裹起来，若是只有一条语句，则可以省略大括号。

有多个参数有返回值，但Lambda体中只有一条返回语句：Comparator comparator = (o1, o2) -> o2 - o1;
对于这种情况，可以省略大括号和return关键字，Lambda体中只剩下需要return的内容
会发现在所有的Lambda表达式中我们并没有为参数定义任何类型，这是因为JVM编译器能够通过上下文自动推断出参数类型。

需要注意的是，并不是所有的接口实现都可以使用Lambda表达式，它需要 函数式接口 的支持，那么什么是函数式接口呢？

函数式接口指的是接口中只有一个抽象方法的接口

这非常好理解，若是接口中存在多个抽象方法，Lambda表达式是无法知晓我们到底需要实现哪个方法的，所以Lambda表达式的使用必须基于函数式接口。

JDK1.8为此专门提供了 @FunctionalInterface 注解来声明一个函数式接口，倘若在接口中声明了该注解，则该接口必须拥有且只能拥有一个方法：

	@FunctionalInterface
	interface calc{
		void  add();
	}
	
若不满足要求则会编译报错。

有些同学可能会发现在使用Lambda表达式实现一些功能时，还需要自己去额外编写一个函数式接口，而事实上，JDK1.8已经为我们内置了四大核心函数式接口，分别是：

Consumer：消费型接口，抽象方法为：void accept(T t);
Supplier：供给型接口，抽象方法为：T get();
Function<T, R>：函数型接口，抽象方法为：R apply(T t);
Predicate：断言型接口，抽象方法为：boolean test(T t);
通过它们就已经能够解决大部分的问题了，具体使用哪个接口可以根据自己的实际需求决定，比如若是需要实现的功能带参数而无返回值，则使用消费型接口；若是需要实现的功能无参数但有返回值，则使用供给型接口。

这里以供给型接口为例，实现一个需求，产生指定个数的整数，并放入集合中，代码如下：

	@Test
	public void test() {
		List<Integer> list = getList(5, () -> new Random().nextInt(20));
		System.out.println(list);
	}

	public List<Integer> getList(int len, Supplier<Integer> supplier) {
		List<Integer> list = new ArrayList<>();
		for (int i = 0; i < len; i++) {
			list.add(supplier.get());
		}
		return list;
	}


### 方法引用

若Lambda体中的内容有方法已经实现了，那么就可以使用方法引用，可以理解为方法引用是Lambda表达式的另一种表现形式，方法引用主要有以下三种形式：

- 对象::实例方法名
- 类::静态方法名
- 类::实例方法名

来看一个例子：

	@Test
	public void test() {
		Consumer<String> consumer = (x)-> System.out.println(x);
		consumer.accept("Hello!");
	}
	
这里使用Consumer消费型接口实现了一个输出字符串的功能，由于Lambda体中的内容已经被 System.out.println 实现了，所以可以简写为：

	@Test
	public void test() {
		Consumer<String> consumer = System.out::println; // 简写为...
		consumer.accept("Hello!");
	}
	
然而方法引用需要遵循一个原则，即：Lambda体中的方法参数和返回值需要与函数式接口中的抽象方法声明一致，比如这里的Consumer接口中的抽象方法为：

void accept(T t);
而输出语句的声明如下：

	public void println(String x) {
		synchronized (this) {
			print(x);
			newLine();
		}
	}
	
它们都带有一个参数且无返回值，所以可以使用方法引用。

又比如：

	@Test
	public void test() {
		User user = new User();
		user.setName("aaa");
		Supplier<String> supplier = () -> user.getName();
		String str = supplier.get();
		System.out.println(str);
	}
	
这里的 user.getName() 也可以使用方法引用，简写为：

Supplier<String> supplier = user::getName;
这也是因为User中get方法与函数式接口中方法参数和返回值的声明相同：

// User对象的getName方法
public String getName() {
    return name;
}

// Supplier接口的get方法
T get();
我们还可以方法引用类的静态方法，例如：

	@Test
	public void test() {
		Comparator<Integer> comparator = Integer::compare;
		int result = comparator.compare(1, 2);
		System.out.println(result);
	}
	
因为Lambda体中的内容已经被compare方法实现且参数和返回值声明与Comparator接口中的抽象方法声明相同，它就能够使用方法引用：

Comparator<Integer> comparator = Integer::compare;
且compare是Integer类的静态方法，这种引用方式被称为类的静态方法引用。

最后一种形式是类的实例方法引用，比如：

	@Test
	public void test() {
		BiPredicate<String, String> biPredicate = (s1, s2) -> s1.equals(s2);
		boolean flag = biPredicate.test("abc", "abc");
		System.out.println(flag);
	}
	
这里实现了函数式接口的方法使其能够判断两个字符串的内容是否相同，它能够被简写为：

BiPredicate<String, String> biPredicate = String::equals;
对于类的实例方法引用，也有它的要求，必须满足第一个参数是方法的调用者，第二个参数是调用方法的参数才能使用该引用方式。



### 构造器引用



构造器引用与方法引用类似，它通过 类名::new 实现，比如：

	@Test
	public void test() {
		Supplier<User> supplier = ()->new User();
		User user = supplier.get();
		System.out.println(user);
	}
	
此时创建User对象的过程就可以使用构造器引用来简化：

	Supplier<User> supplier = User::new;

我们都知道一个类可以有多个重载的构造器，那么构造器引用调用的是类中的哪个构造器呢？和方法引用类似，我们仍然通过构造器方法与接口中抽象方法参数和返回值的声明来判断调用哪个构造器，这里的Supplier接口中的抽象方法是一个不带参数的方法：

	T get();
	
所以它将调用对象的无参构造方法，又比如：

	@Test
	public void test() {
		Function<String, User> function = User::new;
		User user = function.apply("zhangsan");
		System.out.println(user);
	}
	
来看看Function接口中的抽象方法：

	R apply(T t);

它带有一个参数，所以调用的是User对象带一个参数的构造方法：

	public class User {
		private String name;
		
		public User(String name){
			this.name = name;
		}
	 ......
	}
	
	
## Stream API

JDK8中另一重要的新特性就是Stream API，通过Stream，我们能够在集合数据中进行一些非常复杂的查找、过滤、映射等操作，而且实现起来会非常高效和简单。

@Test
public void test() {
    // 通过集合的stream方法获取流
    List<String> list = new ArrayList<>();
    Stream<String> stream = list.stream();

    // 通过Arrays工具类的stream方法获取流
    Stream<String> stream1 = Arrays.stream(new String[]{"1", "2", "3", "4", "5"});

    // 通过Stream类的of方法获取流
    Stream<Integer> stream2 = Stream.of(1, 2, 3, 4, 5);

    // 创建无限流
    Stream<Integer> stream3 = Stream.iterate(0, (x) -> x + 2);
}
以上是四种获取Stream的方式，最后一种创建的是无限流，也就是说，该流创建的是从0开始，每次加2的无限序列。

使用Stream我们能够很轻松地实现过滤操作，比如获取无限流中前5个元素，代码如下：

@Test
public void test() {
    Stream<Integer> stream = Stream.iterate(0, (x) -> x + 2);
    Stream<Integer> stream1 = stream.limit(5);
    stream1.forEach(System.out::println);
}
通过limit方法即可实现获取前5个元素，这里使用了forEach方法进行遍历，并使用了Lambda表达式，我们一起来复习一下，查看forEach方法的源码：

void forEach(Consumer<? super T> action);
可以看到该方法的参数是一个消费型的函数式接口，其接口的抽象方法是带参而无返回值的，所以若是想输出元素，则可以如此做：

stream1.forEach((i) -> System.out.println(i));
又因为输出语句的参数和返回值与接口抽象方法的声明一致，所以可以使用方法引用，最终简化为：

stream1.forEach(System.out::println);
我们还可以通过Stream类的generate方法生成无限流：

@Test
public void test() {
    Stream.generate(Math::random)
        .limit(5)
        .forEach(System.out::println);
}
generate方法需要的是一个供给型接口，它的抽象方法是不带参而有返回值的，我们通过 Math.random() 方法生成随机数作为返回值，又因为random方法也是带一个参而有返回值的，所以可以使用类的静态方法引用，后续的Stream操作中将涉及大量的Lambda表达式，届时将不再过多介绍Lambda表达式的写法。



### Stream的筛选操作

刚才我们通过limit方法简单地了解到Stream的筛选功能，当然了，Stream的筛选能力可远不止如此，这里介绍四种筛选方法：

- filter
- limit
- skip
- distinct

	@Test
	public void test() {
		List<User> userList = Arrays.asList(
			new User("张三", 20, 0),
			new User("李四", 30, 0),
			new User("王五", 25, 1),
			new User("赵六", 42, 1)
		);
		// 筛选出年龄大于25的用户
		Stream<User> stream = userList.stream()
			.filter((user) -> user.getAge() > 25);
		stream.forEach(System.out::println);
	}
	
通过filter方法，我们能够实现很多的筛选功能，比如这里就可以筛选出年龄大于25的用户，filter方法的参数是一个断言型接口，接收一个参数，返回值为boolean类型，即：为true则满足筛选条件，为false则不满足。

limit方法我们已经使用过了，它用来截断Stream，使得Stream获取从开始位置到指定位置的元素内容。

第三个方法是skip：

	@Test
	public void test() {
		List<User> userList = Arrays.asList(
			new User("张三", 20, 0),
			new User("李四", 30, 0),
			new User("王五", 25, 1),
			new User("赵六", 42, 1)
		);
		// 筛选出年龄大于25的用户
		Stream<User> stream = userList.stream()
			.skip(2);
		stream.forEach(System.out::println);
	}
	
它与limit方法正好相反，skip会丢弃掉从开始位置到指定位置的元素内容，比如上面这段程序便会将 张三 和 李四 用户的信息丢掉。

最后一个方法是distinct：

	@Test
	public void test() {
		List<User> userList = Arrays.asList(
			new User("张三", 20, 0),
			new User("李四", 30, 0),
			new User("王五", 25, 1),
			new User("王五", 25, 1),
			new User("王五", 25, 1),
			new User("赵六", 42, 1)
		);
		// 筛选出年龄大于25的用户
		Stream<User> stream = userList.stream()
			.distinct();
		stream.forEach(System.out::println);
	}
	
该方法用于去除流中重复的元素，需要注意的是该方法依赖于equals和hashCode方法，所以User对象必须重写这两个方法：

	public class User {

		private String name;
		private Integer age;
		private Integer sex;
	 ......
		@Override
		public boolean equals(Object o) {
			if (this == o) return true;
			if (o == null || getClass() != o.getClass()) return false;
			User user = (User) o;
			return Objects.equals(name, user.name) && Objects.equals(age, user.age) && Objects.equals(sex, user.sex);
		}

		@Override
		public int hashCode() {
			return Objects.hash(name, age, sex);
		}
	}
	
	
### Stream的映射操作

Stream中的map方法用于映射操作，它能将元素转换成其它形式或提取信息，接收一个函数作为参数，该函数会被应用到Stream中的每个元素上，并将其映射成为一个新的元素。

	@Test
	public void test() {
		List<String> list = Arrays.asList("aa", "bb", "cc", "dd", "ee");
		list.stream()
			.map(String::toUpperCase)
			.forEach(System.out::println);
	}

map方法的参数是一个函数型接口，Stream中的每个元素都会被应用到该接口方法的实现上，所以每个元素都会被转换成大写字母。

map方法也能用于提取Stream中每个元素的信息：

	@Test
	public void test() {
		List<User> userList = Arrays.asList(
			new User("张三", 20, 0),
			new User("李四", 30, 0),
			new User("王五", 25, 1),
			new User("赵六", 42, 1)
		);
		userList.stream()
			.map(User::getName)
			.forEach(System.out::println);
	}
	
此时所有用户的名字就都被提取出来了。


### Stream的排序操作

Stream中的排序也分为两种，自然排序和自定义排序，首先是自然排序，调用sorted方法即可实现：

	@Test
	public void test() {
		List<Integer> list = Arrays.asList(3, 1, 6, 7, 5, 9);
		list.stream()
			.sorted()
			.forEach(System.out::println);
	}
	
自定义排序就需要传入自己实现的比较器：

	@Test
	public void test() {
		List<User> userList = Arrays.asList(
			new User("张三", 20, 0),
			new User("李四", 20, 0),
			new User("王五", 25, 1),
			new User("赵六", 42, 1)
		);
		userList.stream()
			.sorted((u1, u2) -> {
				if (u1.getAge().equals(u2.getAge())) {
					return u1.getName().compareTo(u2.getName());
				} else {
					return u1.getAge().compareTo(u2.getAge());
				}
			})
			.forEach(System.out::println);
	}
	
该程序段实现了按照年龄对User信息进行排序，若年龄相同，则再按照姓名排序。


### Stream的匹配操作

Stream中提供了丰富的匹配方法用于校验数据，分别如下：

- allMatch：是否匹配所有元素
- anyMatch：是否至少匹配一个元素
- noneMatch：是否没有匹配所有元素
- findFirst：返回第一个元素
- findAny：返回流中的任意元素
- count：返回元素个数
- max：返回最大值
- min：返回最小值


比如：

	@Test
	public void test() {
		List<User> userList = Arrays.asList(
			new User("张三", 20, 0),
			new User("李四", 20, 0),
			new User("王五", 25, 1),
			new User("赵六", 42, 1)
		);
		boolean flag = userList.stream()
			.allMatch((u) -> u.getSex().equals(1));
		System.out.println(flag);
	}
	
这段程序中，它会判断Stream中的所有User对象的sex值是否为1，若满足，则返回true，否则返回false，这里的结果当然就是false了。

若是将匹配方法修改为 anyMatch ，则结果会变为true：

	@Test
	public void test() {
		List<User> userList = Arrays.asList(
			new User("张三", 20, 0),
			new User("李四", 20, 0),
			new User("王五", 25, 1),
			new User("赵六", 42, 1)
		);
		boolean flag = userList.stream()
			.anyMatch((u) -> u.getSex().equals(1));
		System.out.println(flag);
	}
	
当所有User对象的sex为0时结果才为false。

noneMatch方法：

	@Test
	public void test() {
		List<User> userList = Arrays.asList(
			new User("张三", 20, 0),
			new User("李四", 20, 0),
			new User("王五", 25, 1),
			new User("赵六", 42, 1)
		);
		boolean flag = userList.stream()
			.noneMatch((u) -> u.getSex().equals(1));
		System.out.println(flag);
	}
	
因为Stream中有User对象的sex值为1，所以没有匹配所有元素是不成立的，故结果为false，只有当所有User对象的sex值均为0，此时没有匹配所有元素成立，结果才为true。

findFirst方法：

	@Test
	public void test() {
		List<User> userList = Arrays.asList(
			new User("张三", 20, 0),
			new User("李四", 20, 0),
			new User("王五", 25, 1),
			new User("赵六", 42, 1)
		);
		Optional<User> optional = userList.stream()
			.sorted(Comparator.comparing(User::getAge))
			.findFirst();
		User user = optional.get();
		System.out.println(user);
	}

该程序段对Stream中的User对象按年龄进行升序，并使用findFirst方法获取第一个User对象，注意这里的返回值是Optional，这也是JDK1.8的新特性，它是用来避免频繁出现的空指针异常的，这个我们后面会介绍。

findAny方法：

	@Test
	public void test() {
		List<User> userList = Arrays.asList(
			new User("张三", 20, 0),
			new User("李四", 20, 0),
			new User("王五", 25, 1),
			new User("赵六", 42, 1)
		);
		Optional<User> optional = userList.stream()
			.filter((u) -> u.getAge().equals(20))
			.findAny();
		User user = optional.get();
		System.out.println(user);
	}
	
这里首先使用filter过滤出年龄为20的用户，此时张三和李四都符合条件，而findAny方法便会从这两个对象中随机选择一个返回，然而这种情况它只会一直返回姓名为张三的User对象，因为当前的Stream是串行流，我们需要获取并行流才能实现随机获取的效果：

	// parallelStream()方法获取并行流
	Optional<User> optional = userList.parallelStream()
		.filter((u) -> u.getAge().equals(20))
		.findAny();
		
		
count方法：

	@Test
	public void test() {
		List<User> userList = Arrays.asList(
			new User("张三", 20, 0),
			new User("李四", 20, 0),
			new User("王五", 25, 1),
			new User("赵六", 42, 1)
		);
		long count = userList.stream()
			.count();
		System.out.println(count);
	}
	
获取当前Stream中的元素个数，结果为4。

max方法：

	@Test
	public void test() {
		List<User> userList = Arrays.asList(
			new User("张三", 20, 0),
			new User("李四", 20, 0),
			new User("王五", 25, 1),
			new User("赵六", 42, 1)
		);
		Optional<User> optional = userList.stream()
			.max(Comparator.comparing(User::getAge));
		User user = optional.get();
		System.out.println(user);
	}

该程序段获取的是Stream中年龄最大的User对象。

min方法：

	@Test
	public void test() {
		List<User> userList = Arrays.asList(
			new User("张三", 20, 0),
			new User("李四", 20, 0),
			new User("王五", 25, 1),
			new User("赵六", 42, 1)
		);
		Optional<User> optional = userList.stream()
			.min(Comparator.comparing(User::getAge));
		User user = optional.get();
		System.out.println(user);
	}
	
将调用方法换为min，则它将获取Stream中年龄最小的User对象。



### Stream的收集操作

	@Test
	public void test() {
		List<Integer> list = Arrays.asList(3, 1, 6, 7, 5, 9);
		Integer result = list.stream()
			.reduce(0, Integer::sum);
		System.out.println(result);
	}
	
该程序段中使用了一个新的方法：reduce ，它的作用是将流中的元素按规则反复地结合起来，得到一个值，比如这里的结果就是将流中的元素全部相加得到和，我们先将方法引用展开再说说其原理：

	@Test
	public void test() {
		List<Integer> list = Arrays.asList(3, 1, 6, 7, 5, 9);
		Integer result = list.stream()
			.reduce(0, (i, j) -> i + j);
		System.out.println(result);
	}

reduce会将第一个参数值0作为起始值，其第一次迭代便是将0作为变量i的值，并从流中取出第一个元素作为变量j的值，所以第一次的执行结果为：0 + 1 = 1 ，并将该结果作为变量i的值，从流中取出第二个元素作为变量j的值进行下一次运算，结果为：1 + 3 = 4 ，以此类推，最终得到总和。

Stream提供了collect方法用于收集操作，它可以将那些经过过滤、映射、排序等操作后的Stream数据重新收集起来，成为一个新的集合数据，代码如下：

	@Test
	public void test() {
		List<User> userList = Arrays.asList(
			new User("张三", 20, 0),
			new User("李四", 20, 0),
			new User("王五", 25, 1),
			new User("赵六", 42, 1)
		);
		List<String> nameList = userList.stream()
			.map(User::getName)
			.collect(Collectors.toList());
		System.out.println(nameList);
	}

此时我们便将所有User对象的姓名取出，并收集到了一个新的集合中。Collectors类提供了非常多的静态方法供我们更方便地进行收集，toList、toSet、toMap、toCollection等等，比如将数据收集成Set集合：

	@Test
	public void test() {
		List<User> userList = Arrays.asList(
			new User("张三", 20, 0),
			new User("李四", 20, 0),
			new User("王五", 25, 1),
			new User("王五", 25, 1),
			new User("赵六", 42, 1)
		);
		Set<String> nameSet = userList.stream()
			.map(User::getName)
			.collect(Collectors.toSet());
		System.out.println(nameSet);
	}

则这样便取出了所有User对象的姓名且不重复。若是想将数据收集成HashSet呢？Collectors类中并未提供toHashSet方法，但我们可以通过toCollection方法实现：

	@Test
	public void test() {
		List<User> userList = Arrays.asList(
			new User("张三", 20, 0),
			new User("李四", 20, 0),
			new User("王五", 25, 1),
			new User("王五", 25, 1),
			new User("赵六", 42, 1)
		);
		Set<String> nameSet = userList.stream()
			.map(User::getName)
			.collect(Collectors.toCollection(HashSet::new));
		System.out.println(nameSet);
	}

toCollection方法需要接收一个供给型接口，通过构造器引用创建HashSet对象即可。

Collectors还提供了一些类似SQL的聚合操作，比如求元素个数：

	Long count = userList.stream().map(User::getName).count();

求平均值：

	Double avgAge = userList.stream().collect(Collectors.averagingDouble(User::getAge));

求总和：

	Integer sumAge = userList.stream().collect(Collectors.summingInt(User::getAge));

最大值：

	Optional<User> optional = userList.stream().collect(Collectors.maxBy(Comparator.comparingInt(User::getAge)));
	User user = optional.get();
	
最小值：

	Optional<User> optional = userList.stream().collect(Collectors.minBy(Comparator.comparingInt(User::getAge)));
	User user = optional.get();
	
	
它甚至能够实现分组，比如按照年龄分组：

	@Test
	public void test() {
		List<User> userList = Arrays.asList(
			new User("张三", 20, 0),
			new User("李四", 20, 0),
			new User("王五", 25, 1),
			new User("王五", 25, 1),
			new User("赵六", 42, 1)
		);
		Map<Integer, List<User>> map = userList.stream().collect(Collectors.groupingBy(User::getAge));
		System.out.println(map);
	}

运行结果：

{20=[User{name='张三', age=20, sex=0}, User{name='李四', age=20, sex=0}], 25=[User{name='王五', age=25, sex=1}, User{name='王五', age=25, sex=1}], 42=[User{name='赵六', age=42, sex=1}]}


还能够进行多级分组，比如先按照年龄分组，再按照性别分组：

	@Test
	public void test() {
		List<User> userList = Arrays.asList(
			new User("张三", 20, 0),
			new User("李四", 20, 0),
			new User("王五", 25, 1),
			new User("王五", 25, 1),
			new User("赵六", 42, 1)
		);
		Map<Integer, Map<String, List<User>>> map = userList.stream().collect(Collectors.groupingBy(User::getAge, Collectors.groupingBy((u) -> {
				if (u.getSex() == 0) {
					return "男";
				} else {
					return "女";
				}
			})));
		System.out.println(map);
	}

运行结果：

{20={男=[User{name='张三', age=20, sex=0}, User{name='李四', age=20, sex=0}]}, 25={女=[User{name='王五', age=25, sex=1}, User{name='王五', age=25, sex=1}]}, 42={女=[User{name='赵六', age=42, sex=1}]}}
按年龄进行分区：

	@Test
	public void test() {
		List<User> userList = Arrays.asList(
			new User("张三", 20, 0),
			new User("李四", 20, 0),
			new User("王五", 25, 1),
			new User("王五", 25, 1),
			new User("赵六", 42, 1)
		);
		Map<Boolean, List<User>> map = userList.stream().collect(Collectors.partitioningBy((u) -> u.getAge() > 20));
		System.out.println(map);
	}

运行结果：

{false=[User{name='张三', age=20, sex=0}, User{name='李四', age=20, sex=0}], true=[User{name='王五', age=25, sex=1}, User{name='王五', age=25, sex=1}, User{name='赵六', age=42, sex=1}]}
它会按照年龄大于20和小于等于20的规则将User对象分为两组。

 
## Optional

Optional类是一个容器类，代表一个值存在或者不存在，JDK1.8中使用Optional来更好地表示一个null值，可以大大地避免空指针异常的产生。

该类下有一些常用的方法：

- of(T t)：创建Optional实例
- empty()：创建一个空的Optional实例
- ofNullable(T t)：若t不为null，创建Optional实例，否则创建空的Optional实例
- isPresent()：判断是否包含值
- orElse(T t)：如果调用对象包含值，返回该值，否则返回t
- orElseGet(Suppiler s)：如果调用对象包含值，返回该值，否则返回s获取的值
- map(Function f)：如果有值对其处理，并返回处理后的Optional，否则返回空的Optional实例

	@Test
	public void test() {
		Optional<User> optional = Optional.of(new User());
		User user = optional.get();
		System.out.println(user);
	}

需要注意，of方法不能接收一个为null的参数，否则仍然会报空指针异常，此时应该使用 ofNullable 方法来处理：

	@Test
	public void test() {
		Optional<Object> optional = Optional.ofNullable(null);
		System.out.println(optional.get());
	}
	
它不会报空指针异常，随之而来的是NoSuchElementException异常：

	java.util.NoSuchElementException: No value present
	 at java.util.Optional.get(Optional.java:135)
		at com.wwj.test.TestDemo.test(TestDemo.java:15)
	
	
其它方法的使用非常简单，这里直接列出：

@Test
public void test() {
    Optional<User> optional = Optional.of(new User("张三", 20, 0));
    // 获取值
    User user = optional.get();
    // 判断是否有值，结果为true
    boolean flag = optional.isPresent();
    // 若optional中有值，则返回该值，否则返回orElse新创建的值
    User user1 = optional.orElse(new User("李四", 22, 0));
    // 该方法与orElse功能相同，区别在于该方法的参数是一个供给型接口，通过接口实现可以编写更加复杂的功能
    User user2 = optional.orElseGet(() -> new User("李四", 22, 0));
    // 若Optional中有值，则返回处理后的结果，否则返回一个Optional.empty实例
    Optional<String> str = optional.map(User::getName);
    // 该方法与map方法功能相同，区别在于该方法的返回值必须是Optional类型
    Optional<String> str2 = optional.flatMap((u) -> Optional.of(u.getName()));
}


## 接口可以有默认方法和静态方法了

	public class TestDemo {
		@Test
		public void test() {
			TestInterface testInterface = new TestInterface();
			int result = testInterface.calc(1, 2);
			System.out.println(result);
		}
	}

	interface MyInterface {
		default int calc(int x, int y) {
			return x + y;
		}
	}

	class TestInterface implements MyInterface {

	}

JDK1.8以后，接口中可以提供已实现的默认方法了，此时类在实现该接口时就无需实现默认方法了，但有这么一种情况：

	public class TestDemo {

		@Test
		public void test() {
			TestClass testClass = new TestClass();
			int result = testClass.calc(1, 2);
			System.out.println(result);
		}
	}

	class TestClass extends MyClass implements MyInterface{

	}

	interface MyInterface {
		default int calc(int x, int y) {
			return x + y;
		}
	}

	class MyClass {
		public int calc(int x, int y) {
			return x - y;
		}
	}
	
TestClass类分别继承了MyClass类和实现了MyInterface接口，然而MyClass类和MyInterface接口中都有一个相同的方法，此时调用TestClass对象的calc方法，执行的究竟是谁的方法呢？

事实上，JDK1.8规定，接口的默认方法遵循 类优先 的原则，即：一个接口中定义了一个默认方法，而另外一个父类或者接口中又定义了一个相同的方法，那么它会优先选择父类中的方法，而忽略掉接口中的默认方法。

若是两个接口中都有一个相同的默认方法，则某个类在同时实现这两个接口的时候便会发生冲突，此时实现类就必须覆盖方法来解决冲突：

	class TestClass implements MyInterface,MyInterfac2{

		@Override
		public int calc(int x, int y) {
			return x + y;
		}
	}

	interface MyInterface {
		default int calc(int x, int y) {
			return x + y;
		}
	}

	interface MyInterfac2 {
		default int calc(int x, int y) {
			return x + y;
		}
	}
	
	
除了默认方法，JDK1.8中还可以拥有静态方法：

	public class TestDemo {

		@Test
		public void test() {
			MyInterface.show();
		}
	}

	interface MyInterface {
		static void show() {
			System.out.println("Hello!");
		}
	}
	

	
## 全新的日期时间API

传统的日期时间API大部分已经过期，比如Date类中的方法，而且方法比较难用，传参复杂，多线程环境下还会有安全问题：

	@Test
	public void test() throws ExecutionException, InterruptedException {
		SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
		Callable<Date> callable = () -> format.parse("2021-04-11");
		ExecutorService pool = Executors.newFixedThreadPool(10);
		List<Future<Date>> result = new ArrayList<>();
		for (int i = 0; i < 10; i++) {
			result.add(pool.submit(callable));
		}
		for (Future<Date> dateFuture : result) {
			System.out.println(dateFuture.get());
		}
	}
	
运行结果：

	Sun Apr 11 00:00:00 CST 2021

    java.util.concurrent.ExecutionException: java.lang.NumberFormatException: multiple points
    at com.wwj.test.TestDemo.test(TestDemo.java:23)
	
当然了，解决这一线程安全问题的方法有很多，在每个线程中都新创建SimpleDateFormat对象：

Callable<Date> callable = () -> new SimpleDateFormat("yyyy-MM-dd").parse("2021-04-11");

或者使用ThreadLocal类：

	public class DateFormatThreadLocal {

		private static final ThreadLocal<DateFormat> format = new ThreadLocal<DateFormat>() {
			@Override
			protected DateFormat initialValue() {
				return new SimpleDateFormat("yyyy-MM-dd");
			}
		};

		public static Date convert(String date) throws ParseException {
			return format.get().parse(date);
		}
	}
	
然后使用该类进行日期格式化：

	Callable<Date> callable = () -> DateFormatThreadLocal.convert("2021-04-11");
	
	
## 而JDK1.8提供了一套全新的日期时间API用于简化对日期时间的处理，先来看一个简单示例：

	@Test
	public void test() throws ExecutionException, InterruptedException {
		DateTimeFormatter formatter = DateTimeFormatter.ISO_LOCAL_DATE;
		LocalDate date = LocalDate.parse("2021-04-11", formatter);
		System.out.println(date);
	}
	
通过DateTimeFormatter可以指定转换日期的格式，它内置了非常多的日期格式：Image若是这里没有自己需要的日期格式，也可以自定义：

	DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyyMMdd");
	LocalDate date = LocalDate.parse("20210411", formatter);
	System.out.println(date);

因为这些新的日期时间API都是被final修饰的，所以它们天生就是线程安全的。

### 先来介绍三个提供日期时间的API：

- LocalDate
- LocalTime
- LocalDateTime

它们的用法完全相同，所以这里只介绍LocalDate类的使用：

@Test
public void test() {
    // 获取当前日期的LocalDate实例
    LocalDate date = LocalDate.now();
    System.out.println(date);
    // 获取指定日期的LocalDate实例
    LocalDate date2 = LocalDate.of(2021, 4, 10);
    System.out.println(date2);
    System.out.println("------------");
    System.out.println("一天后的日期:" + date.plusDays(1));
    System.out.println("一星期后的日期:" + date.plusWeeks(1));
    System.out.println("一个月后的日期:" + date.plusMonths(1));
    System.out.println("一年后的日期:" + date.plusYears(1));
    System.out.println("------------");
    System.out.println("一天前的日期:" + date.minusDays(1));
    System.out.println("一星期前的日期:" + date.minusWeeks(1));
    System.out.println("一个月前的日期:" + date.minusMonths(1));
    System.out.println("一年前的日期:" + date.minusYears(1));
    System.out.println("------------");
    System.out.println("获取日:" + date.getDayOfMonth());
    System.out.println("获取月:" + date.getMonthValue());
    System.out.println("获取年:" + date.getYear());
}
运行结果：

2021-04-11
2021-04-10
------------
一天后的日期:2021-04-12
一星期后的日期:2021-04-18
一个月后的日期:2021-05-11
一年后的日期:2022-04-11
------------
一天前的日期:2021-04-10
一星期前的日期:2021-04-04
一个月前的日期:2021-03-11
一年前的日期:2020-04-11
------------
获取日:11
获取月:4
获取年:2021

### 若是想获取时间戳，可以使用Instant类：

@Test
public void test() throws InterruptedException {
    Instant instant = Instant.now();
    // 默认是UTC时区
    System.out.println(instant);
    // 向后偏移8个小时即为我国的时间
    OffsetDateTime offsetDateTime = instant.atOffset(ZoneOffset.ofHours(8));
    System.out.println(offsetDateTime);
    // 获取时间戳的毫秒数
    System.out.println(instant.toEpochMilli());
    // 获取向后偏移1个小时的时间戳，时间戳默认以1970年1月1日00:00:00开始
    Instant instant1 = Instant.ofEpochSecond(60 * 60);
    System.out.println(instant1);
    // 计算两个时间戳之间的间隔
    Instant instant2 = Instant.now();
    Thread.sleep(1000);
    Instant instant3 = Instant.now();
    Duration duration = Duration.between(instant2, instant3);
    System.out.println(duration);
    // 计算两个时间之间的间隔
    LocalTime localTime = LocalTime.now();
    Thread.sleep(1000);
    LocalTime localTime2 = LocalTime.now();
    Duration duration1 = Duration.between(localTime, localTime2);
    System.out.println(duration1);
    // 计算两个日期之间的间隔
    LocalDate localDate = LocalDate.now();
    LocalDate localDate1 = LocalDate.of(2021, 4, 9);
    Period period = Period.between(localDate, localDate1);
    System.out.println(period);
}
运行结果：

2021-04-11T09:53:33.836Z
2021-04-11T17:53:33.836+08:00
1618134813836
1970-01-01T01:00:00Z
PT1.012S
PT1.005S
P-2D


### 为了更加方便地操作日期时间，JDK1.8还为我们提供了时间校正器 TemporalAdjuster ，通过它我们就能够非常方便地操纵日期和时间：

@Test
public void test() {
    LocalDateTime dateTime = LocalDateTime.now();
    System.out.println(dateTime);
    // 计算下一个周日
    LocalDateTime nextSunday = dateTime.with(TemporalAdjusters.next(DayOfWeek.SUNDAY));
    System.out.println(nextSunday);
    // 自定义日期标识，计算下一个工作日
    LocalDateTime localDateTime = dateTime.with((d) -> {
        LocalDateTime ldt = (LocalDateTime) d;
        DayOfWeek dayOfWeek = ldt.getDayOfWeek();
        if (dayOfWeek.equals(DayOfWeek.FRIDAY)) {
            return ldt.plusDays(3); // 如果是周五，则三天后就是工作日
        } else if (dayOfWeek.equals(DayOfWeek.SATURDAY)) {
            return ldt.plusDays(2); // 如果是周六，则两天后就是工作日
        } else {
            return ldt.plusDays(1); // 其它时间都加一天
        }
    });
    System.out.println(localDateTime);
}


### 最后是时间格式化操作，JDK1.8中使用DateTimeFormatter类来格式化日期和时间：

@Test
public void test() {
    // 使用内置的日期格式
    DateTimeFormatter formatter = DateTimeFormatter.ISO_DATE;
    LocalDateTime localDateTime = LocalDateTime.now();
    String strDate = localDateTime.format(formatter);
    System.out.println(strDate);
    // 使用自定义格式
    DateTimeFormatter formatter2 = DateTimeFormatter.ofPattern("yyyy年MM月dd日 HH时mm分ss秒");
    String strDate2 = localDateTime.format(formatter2);
    System.out.println(strDate2);
    // 将日期时间字符串解析成LocalDateTime实例
    LocalDateTime parse = localDateTime.parse(strDate2, formatter2);
    System.out.println(parse);
}


### 还有时区的操作：

@Test
public void test() {
    // 获取所有支持的时区
    Set<String> set = ZoneId.getAvailableZoneIds();
    // 在获取日期时设置时区
    LocalDateTime localDateTime = LocalDateTime.now(ZoneId.of("America/Cuiaba"));
    System.out.println(localDateTime);
    LocalDateTime localDateTime2 = LocalDateTime.now();
    // 获取日期后设置时区
    ZonedDateTime zonedDateTime = localDateTime2.atZone(ZoneId.of("Asia/Shanghai"));
    System.out.println(zonedDateTime);
}

运行结果：

	2021-04-11T08:00:44.308
	2021-04-11T20:00:44.334+08:00[Asia/Shanghai]
