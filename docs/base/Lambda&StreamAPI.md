# Lambda表达式

**举例**

```java
 (o1,o2) -> Integer.compare(o1,o2);
```



**格式**

- -> :lambda操作符 或 箭头操作符

- ->左边：lambda形参列表 （其实就是接口中的抽象方法的形参列表）

- ->右边：lambda体 （其实就是重写的抽象方法的方法体）

##  1、Lambda表达式的使用：（分为6种情况介绍）

```java
public class LambdaTest1 {
    //语法格式一：无参，无返回值
    @Test
    public void test1(){
        Runnable r1 = new Runnable() {
            @Override
            public void run() {
                System.out.println("我爱北京天安门");
            }
        };
		//
        Runnable r2 = () -> {
            System.out.println("我爱北京故宫");
        };
    }
````
//
```java
    //语法格式二：Lambda 需要一个参数，但是没有返回值。
    @Test
    public void test2(){
        Consumer<String> con = new Consumer<String>() {
            @Override
            public void accept(String s) {
                System.out.println(s);
            }
        };
        //
        Consumer<String> con1 = (String s) -> {
            System.out.println(s);
        };
    }
````
//
```java
    //语法格式三：数据类型可以省略，因为可由编译器推断得出，称为“类型推断”
    @Test
    public void test3(){
        Consumer<String> con1 = (String s) -> {
            System.out.println(s);
        };
        //
        Consumer<String> con2 = (s) -> {
            System.out.println(s);
        };
    }
````
//
```java
    //语法格式四：Lambda 若只需要一个参数时，参数的小括号可以省略
    @Test
    public void test4(){
        Consumer<String> con1 = (s) -> {
            System.out.println(s);
        };
      	//
        Consumer<String> con2 = s -> {
            System.out.println(s);
        };
    }
````
//
```java
    //语法格式五：Lambda 需要两个或以上的参数，多条执行语句，并且可以有返回值
    @Test
    public void test5(){
        Comparator<Integer> com1 = new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                return o1.compareTo(o2);
            }
        };
		//
        Comparator<Integer> com2 = (o1,o2) -> {
            return o1.compareTo(o2);
        };
    }
````
//
```java
    // 语法格式六：当 Lambda 体只有一条语句时，return 与大括号若有，都可以省略
    // 省略{}后，return必须拿掉，否则不能省略
    @Test
    public void test6(){
        Comparator<Integer> com1 = (o1,o2) -> {
            return o1.compareTo(o2);
        };
		//
        Comparator<Integer> com2 = (o1,o2) -> o1.compareTo(o2);
    }
}

```



**总结**

 *    ->左边：lambda形参列表的参数类型可以省略(类型推断)；如果lambda形参列表只有一个参数，其一对()也可以省略；省略{}后，return必须拿掉，否则不能省略
 *    ->右边：lambda体应该使用一对{}包裹；如果lambda体只有一条执行语句（可能是return语句），省略这一对{}和return关键字


> Lambda 表达式的本质：作为函数式接口的实例

## 2、函数式接口

**如果一个接口中，只声明了一个抽象方法**，则此接口就称为函数式接口。

可以在一个接口上使用 @FunctionalInterface 注解，这样做可以检查它是否是一个函数式接口。

> 以前用匿名实现类表示的现在都可以用Lambda表达式来写



### 内置四大核心函数式接口

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204211555124.png" alt="image-20220421155539918" style="zoom:50%;" />

**其他接口**

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204211556577.png" alt="image-20220421155621312" style="zoom:50%;" />



## 3、方法引用

- 当要传递给Lambda体的操作，已经有实现的方法了，可以使用方法引用!
- **要求**：实现接口的抽象方法的参数列表和返回值类型，必须与方法引用的方法的参数列表和返回值类型保持一致!（仅针对情况1和2）

### 对象::实例方法名

```java
// 情况一：对象 :: 实例方法
//Consumer中的void accept(T t)
//PrintStream中的void println(T t)
@Test
public void test1() {
    Consumer<String> con1 = str -> System.out.println(str);
    con1.accept("北京");

    System.out.println("*******************");
    PrintStream ps = System.out;
    Consumer<String> con2 = ps::println;
    con2.accept("beijing");
}

//Supplier中的T get()
//Employee中的String getName()
@Test
public void test2() {
    Employee emp = new Employee(1001,"Tom",23,5600);

    Supplier<String> sup1 = () -> emp.getName();
    System.out.println(sup1.get());

    System.out.println("*******************");
    Supplier<String> sup2 = emp::getName;
    System.out.println(sup2.get());

}
```



### 类::静态方法名

```java
// 情况二：类 :: 静态方法
//Comparator中的int compare(T t1,T t2)
//Integer中的int compare(T t1,T t2)
@Test
public void test3() {
    Comparator<Integer> com1 = (t1,t2) -> Integer.compare(t1,t2);
    System.out.println(com1.compare(12,21));

    System.out.println("*******************");

    Comparator<Integer> com2 = Integer::compare;
    System.out.println(com2.compare(12,3));

}

//Function中的R apply(T t)
//Math中的Long round(Double d)
@Test
public void test4() {
    Function<Double,Long> func = new Function<Double, Long>() {
        @Override
        public Long apply(Double d) {
            return Math.round(d);
        }
    };

    System.out.println("*******************");

    Function<Double,Long> func1 = d -> Math.round(d);
    System.out.println(func1.apply(12.3));

    System.out.println("*******************");

    Function<Double,Long> func2 = Math::round;
    System.out.println(func2.apply(12.6));
}

```



### 类::实例方法名

> 当函数式接口方法的第一个参数是需要引用方法的调用者，并且第二个参数是需要引用方法的参数(或无参数)时：`ClassName::methodName`

```java
// 情况三：类 :: 实例方法  (有难度)
// Comparator中的int comapre(T t1,T t2)
// String中的int t1.compareTo(t2)
@Test
public void test5() {
    Comparator<String> com0 = new Comparator<String>() {
        @Override
        public int compare(String o1, String o2) {
            return o1 - o2;
        }
    };
    Comparator<String> com1 = (s1,s2) -> s1.compareTo(s2);
    System.out.println(com1.compare("abc","abd"));

    System.out.println("*******************");

    Comparator<String> com2 = String :: compareTo;
    System.out.println(com2.compare("abd","abm"));
}

// Function中的R apply(T t)
// Employee中的String getName();
@Test
public void test7() {
    Employee employee = new Employee(1001, "Jerry", 23, 6000);

    Function<Employee,String> func1 = e -> e.getName();
    System.out.println(func1.apply(employee));

    System.out.println("*******************");

    Function<Employee,String> func2 = Employee::getName;
    System.out.println(func2.apply(employee));

}
```



## 4、构造器引用

**格式**

ClassName :: new

**要求**

构造器参数列表要与接口中抽象方法的参数列表一致!

且方法的返回值为构造器对应类的对象。

```java
//构造器引用
//Supplier中的T get()
//Employee的空参构造器：Employee()
@Test
public void test1(){

    Supplier<Employee> sup = new Supplier<Employee>() {
        @Override
        public Employee get() {
            return new Employee();
        }
    };

    Supplier<Employee>  sup1 = () -> new Employee();

    Supplier<Employee>  sup2 = Employee :: new;
}
```

//

```java
//数组引用
//Function中的R apply(T t)
@Test
public void test4(){
    Function<Integer,String[]> func1 = length -> new String[length];
    String[] arr1 = func1.apply(5);

    Function<Integer,String[]> func2 = String[] :: new;
    String[] arr2 = func2.apply(10);
}
```



# Stream API

使用 Stream API 对集合数据进行操作，就类似于使用 SQL 执行的数据库查询。

① Stream 自己不会存储元素。
② Stream 不会改变源对象。相反，他们会返回一个持有结果的新Stream。
③ Stream 操作是延迟执行的。这意味着他们会等到需要结果的时候才执行



















```java
return stack.stream().mapToInt(i->i).toArray();
```

