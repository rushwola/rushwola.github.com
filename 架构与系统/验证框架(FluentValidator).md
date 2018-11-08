---
title: 验证框架(FluentValidator)
tags: FluentValidator
grammar_cjkRuby: true
---
1 背景

在互联网行业中，基于Java开发的业务类系统，不管是服务端还是客户端，业务逻辑代码的更新往往是非常频繁的，这源于功能的快速迭代特性。在一般公司内部，特别是使用Java web技术构建的平台中，不管是基于模块化还是服务化的，业务逻辑都会相对复杂。

这些系统之间、系统内部往往存在大量的API接口，这些接口一般都需要对入参（输入参数的简称）做校验，以保证：
1） 核心业务逻辑能够顺利按照预期执行。
2） 数据能够正常存取。
3） 数据安全性。包括符合约束以及限制，有访问权限控制以及不出现SQL注入等问题。

开发人员在维护核心业务逻辑的同时，还需要为输入做严格的校验。当输入不合法时，能够给caller一个明确的反馈，最常见的反馈就是返回封装了result的对象或者抛出exception。

一些常见的验证代码片段如下所示：

public Response execute(Request request) {
    if (request == null) {
        throw BizException();
    }

    List cars = request.getCars();
    if (CollectionUtils.isEmpty(cars)) {
        throw BizException();
    }

    for (Car car : cars) {
        if (car.getSeatCount() < 2) {
            throw BizException(); 
        }
    }

    // do core business logic
}

我们不能说这是反模式（anti-pattern），但是从中我们可以发现，它不够优雅而且违反一些范式：
1）违反单一职责原则（Single responsibility）。核心业务逻辑（core business logic）和验证逻辑（validation logic）耦合在一个类中。
2）开闭原则（Open/closed）。我们应该对扩展开放，对修改封闭，验证逻辑不好扩展，而且一旦需要修改需要动整体这个类。
3）DRY原则（Don’t repeat yourself）。代码冗余，相同逻辑可能散落多处，长此以往不好收殓。
2 为何要使用FluentValidator

原因很简单，第一为了优雅，出色的程序员都有点洁癖，都希望让验证看起来很舒服；第二，为了尽最大可能符合这些优秀的原则，做clean code。

FluentValidator就是这么一个工具类库，适用于以Java语言开发的程序，让开发人员回归focus到业务逻辑上，使用流式（Fluent Interface）调用风格让验证跑起来很优雅，同时验证器（Validator）可以做到开闭原则，实现最大程度的复用。
3 FluentValidator特点

这里算是Quick learn了解下，也当且看做打广告吧，看了这些特点，希望能给你往下继续阅读的兴趣：）

1) 验证逻辑与业务逻辑不再耦合
摒弃原来不规范的验证逻辑散落的现象。

2) 校验器各司其职，好维护，可复用，可扩展
一个校验器（Validator）只负责某个属性或者对象的校验，可以做到职责单一，易于维护，并且可复用。

3) 流式风格（Fluent Interface）调用
借助Martin大神提倡的流式API风格，使用“惰性求值（Lazy evaluation）”式的链式调用，类似guava、Java8 stream API的使用体验。

4) 使用注解方式验证
可以装饰在属性上，减少硬编码量。

5) 支持JSR 303 – Bean Validation标准
或许你已经使用了Hibernate Validator，不用抛弃它，FluentValidator可以站在巨人的肩膀上。

6) Spring良好集成
校验器可以由Spring IoC容器托管。校验入参可以直接使用注解，配置好拦截器，核心业务逻辑完全没有验证逻辑的影子，干净利落。

7) 回调给予你充分的自由度
验证过程中发生的错误、异常，验证结果的返回，开发人员都可以定制。

4 哪里可以获取到FluentValidator

项目托管在github上，地址点此https://github.com/neoremind/fluent-validator。说明文档全英完成，i18n化，同时使用Apache2 License开源

5 上手
5.1 maven引入依赖

添加如下依赖到maven的pom.xml文件中：

``` stylus
<dependency>
    <groupId>com.baidu.unbiz</groupId>
    <artifactId>fluent-validator</artifactId>
    <version>1.0.5</version>
</dependency>
```
上面这个FluentValidator是个基础核心包，只依赖于slf4j和log4j，如果你使用logback，想去掉log4j，排除掉的方法如下所示：

``` stylus
<dependency>
    <groupId>com.baidu.unbiz</groupId>
    <artifactId>fluent-validator</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```
5.2 开发业务领域模型

从广义角度来说DTO（Data Transfer Object）、VO（Value Object）、BO（Business Object）、POJO等都可以看做是业务表达模型。

我们这里创建一个汽车类（Car）的POJO，里面定义了牌照（license plate）、座椅数（seat count）、生产商（manufacturer）。

``` stylus
public class Car {
    private String manufacturer;
    private String licensePlate;
    private int seatCount;

    // getter and setter...
}
```

5.3 开发一个专职的Validator

实际这里需要开发三个Validator，分别对Car的3个属性进行校验，这里以座椅数为例展示如何开发一个Validator，其他两个省略。

``` stylus
public class CarSeatCountValidator extends ValidatorHandler<Integer> implements Validator<Integer> {

    @Override
    public boolean validate(ValidatorContext context, Integer t) {
        if (t < 2) {
            context.addErrorMsg(String.format("Seat count is not valid, invalid value=%s", t));
            return false;
        }
        return true;
    }
}
```
很简单，实现Validator接口，泛型T规范这个校验器待验证的对象的类型，继承ValidatorHandler可以避免实现一些默认的方法，例如accept()，后面会提到，validate()方法第一个参数是整个校验过程的上下文，第二个参数是待验证对象，也就是座椅数。

验证逻辑很简单，座椅数必须大于1，否则通过context放入错误消息并且返回false，成功返回true。

5.4 开始验证吧

二话不说，直接上代码：

``` stylus
Car car = getCar();

Result ret = FluentValidator.checkAll()
                            .on(car.getLicensePlate(), new CarLicensePlateValidator())
                            .on(car.getManufacturer(), new CarManufacturerValidator())
                            .on(car.getSeatCount(), new CarSeatCountValidator())
                            .doValidate()
                            .result(toSimple());

System.out.println(ret);
```
我想不用多说，如果你会英文，你就能知道这段代码是如何工作的。这就是流式风格（Fluent Interface）调用的优雅之处，让代码更可读、更好理解。

还是稍微说明下，首先我们通过FluentValidator.checkAll()获取了一个FluentValidator实例，紧接着调用了failFast()表示有错了立即返回，它的反义词是failOver，然后，一连串on()操作表示在Car的3个属性上依次使用3个校验器进行校验（这个过程叫做applying constraints），截止到此，真正的校验还并没有做，这就是所谓的“惰性求值（Lazy valuation）”，有点像Java8 Stream API中的filter()、map()方法，直到doValidate()验证才真正执行了，最后我们需要收殓出来一个结果供caller获取打印，直接使用默认提供的静态方法toSimple()来做一个回调函数传入result()方法，最终返回Result类，如果座椅数不合法，那么控制台打印结果如下：

``` stylus
Result{isSuccess=false, errors=[Seat count is not valid, invalid value=99]}
```
6 深入实践
6.1 Validator详解

Validator接口定义如下：

``` stylus
public interface Validator<T> {

 /**  * 判断在该对象上是否接受或者需要验证  * <p/>  * 如果返回true，那么则调用{@link #validate(ValidatorContext, Object)}，否则跳过该验证器  *  * @param context 验证上下文  * @param t 待验证对象  *  * @return 是否接受验证  */
 boolean accept(ValidatorContext context, T t);

 /**  * 执行验证  * <p/>  * 如果发生错误内部需要调用{@link ValidatorContext#addErrorMsg(String)}方法，也即<code>context.addErrorMsg(String)  * </code>来添加错误，该错误会被添加到结果存根{@link Result}的错误消息列表中。  *  * @param context 验证上下文  * @param t 待验证对象  *  * @return 是否验证通过  */
 boolean validate(ValidatorContext context, T t);

 /**  * 异常回调  * <p/>  * 当执行{@link #accept(ValidatorContext, Object)}或者{@link #validate(ValidatorContext, Object)}发生异常时的如何处理  *  * @param e 异常  * @param context 验证上下文  * @param t 待验证对象  */
 void onException(Exception e, ValidatorContext context, T t);

}
```
ValidatorHandler是实现Validator接口的一个模板类，如果你自己实现的Validator不想覆盖上面3个方法，可以继承这个ValidatorHandler。

``` stylus
public class ValidatorHandler<T> implements Validator<T> {

    @Override
    public boolean accept(ValidatorContext context, T t) {
        return true;
    }

    @Override
    public boolean validate(ValidatorContext context, T t) {
        return true;
    }

    @Override
    public void onException(Exception e, ValidatorContext context, T t) {

    }
}
```
内部校验逻辑发生错误时候，有两个处理办法，

第一，简单处理，直接放入错误消息。

``` stylus
context.addErrorMsg("Something is wrong about the car seat count!");
return false;
```
第二，需要详细的信息，包括错误消息，错误属性/字段，错误值，错误码，都可以自己定义，放入错误的方法如下，create()方法传入消息（必填），setErrorCode()方法设置错误码（选填），setField()设置错误字段（选填），setInvalidValue()设置错误值（选填）。当然这些信息需要result(toComplex())才可以获取到，详见6.7小节。

``` stylus
context.addError(ValidationError.create("Something is wrong about the car seat count!").setErrorCode(100).setField("seatCount").setInvalidValue(t));
return false;
```
6.2 ValidatorChain

on()的一连串调用实际就是构建调用链，因此理所当然可以传入一个调用链

``` stylus
ValidatorChain chain = new ValidatorChain();
List<Validator> validators = new ArrayList<Validator>();
validators.add(new CarValidator());
chain.setValidators(validators);

Result ret = FluentValidator.checkAll().on(car, chain).doValidate().result(toSimple()
```
6.3 onEach

如果要验证的是一个集合（Collection）或者数组，那么可以使用onEach，FluentValidator会自动为你遍历，依次apply constraints。

``` stylus
FluentValidator.checkAll()
               .onEach(Lists.newArrayList(new Car(), new Car()), new CarValidator());
FluentValidator.checkAll()
               .onEach(new Car[]{}, new CarValidator());
```

6.4 fail fast or fail over

当出现校验失败时，也就是Validator的validate()方法返回了false，那么是继续还是直接退出呢？默认为使用failFast()方法，直接退出，如果你想继续完成所有校验，使用failOver()来skip掉。

FluentValidator.checkAll().failFast()
               .on(car.getManufacturer(), new CarManufacturerValidator());
FluentValidator.checkAll().failOver()
               .on(car.getManufacturer(), new CarManufacturerValidator());

6.5 when

on()后面可以紧跟一个when()，当when满足expression表达式on才启用验证，否则skip调用。

FluentValidator.checkAll()
               .on(car.getManufacturer(), new CarManufacturerValidator()).when(a == b)

6.6 验证回调callback

doValidate()方法接受一个ValidateCallback接口，接口定义如下：

public interface ValidateCallback {

 /**  * 所有验证完成并且成功后  *  * @param validatorElementList 验证器list  */
 void onSuccess(ValidatorElementList validatorElementList);

 /**  * 所有验证步骤结束，发现验证存在失败后  *  * @param validatorElementList 验证器list  * @param errors 验证过程中发生的错误  */
 void onFail(ValidatorElementList validatorElementList, List<ValidationError> errors);

 /**  * 执行验证过程中发生了异常后  *  * @param validator 验证器  * @param e 异常  * @param target 正在验证的对象  *  * @throws Exception  */
 void onUncaughtException(Validator validator, Exception e, Object target) throws Exception;

}

默认的，使用不含参数的doValidate()方法，FluentValidator使用DefaultValidateCallback，其实现如下，可以看出出错了，成功了什么也不做，有不可控异常的时候直接抛出。

public class DefaultValidateCallback implements ValidateCallback {

    @Override
    public void onSuccess(ValidatorElementList validatorElementList) {
    }

    @Override
    public void onFail(ValidatorElementList validatorElementList, List<ValidationError> errors) {
    }

    @Override
    public void onUncaughtException(Validator validator, Exception e, Object target) throws Exception {
        throw e;
    }
}

如果你不满意这种方式，例如成功的时候打印一些消息，一种实现方式如下：

Result ret = FluentValidator.checkAll()
                            .on(car.getSeatCount(), new CarSeatCountValidator())
                            .doValidate(new DefaulValidateCallback() {

                                @Override
                                public void onSuccess(ValidatorElementList validatorElementList) {
                                    LOG.info("all ok!");
                                }
                            }).result(toSimple());

6.7 获取结果

result()接受一个ResultCollector接口，如上面所示，toSimple()实际是个静态方法，这种方式在Java8 Stream API中很常见，默认可以使用FluentValidator自带的简单结果Result，如果需要可以使用复杂ComplexResult，内含错误消息，错误属性/字段，错误值，错误码，如下所示：

ComplexResult ret = FluentValidator.checkAll().failOver()
                                   .on(company, new CompanyCustomValidator())
                                   .doValidate().result(toComplex());

当然，如果你想自己实现一个结果类型，完全可以定制，实现ResultCollector接口即可。

public interface ResultCollector<T> {

 /**  * 转换为对外结果  *  * @param result 框架内部验证结果  *  * @return 对外验证结果对象  */
 T toResult(ValidationResult result);
}

6.8 关于RuntimeValidateException

如果验证中发生了一些不可控异常，例如数据库调用失败，RPC连接失效等，会抛出一些异常，如果Validator没有try-catch处理，FluentValidator会将这些异常封装在RuntimeValidateException，然后再re-throw出去，这个情况你应该知晓并作出你认为最正确的处理。
6.9 context上下文共享

通过putAttribute2Context()方法，可以往FluentValidator注入一些键值对，在所有Validator中共享，有时候这相当有用。

例如下面展示了在caller添加一个ignoreManufacturer属性，然后再Validator中拿到这个值的过程。

FluentValidator.checkAll()
               .putAttribute2Context("ignoreManufacturer", true)
               .on(car.getManufacturer(), new CarManufacturerValidator())
               .doValidate().result(toSimple());

public class CarManufacturerValidator extends ValidatorHandler<String> implements Validator<String> {

    @Override
    public boolean validate(ValidatorContext context, String t) {
        Boolean ignoreManufacturer = context.getAttribute("ignoreManufacturer", Boolean.class);
        if (ignoreManufacturer != null && ignoreManufacturer) {
            return true;
        }
        // ...
    }

}

6.10 闭包

通过putClosure2Context()方法，可以往FluentValidator注入一个闭包，这个闭包的作用是在Validator内部可以调用，并且缓存结果到Closure中，这样caller在上层可以获取这个结果。

典型的应用场景是，当需要频繁调用一个RPC的时候，往往该执行线程内部一次调用就够了，多次调用会影响性能，我们就可以缓存住这个结果，在所有Validator间和caller中共享。

下面展示了在caller处存在一个manufacturerService，它假如需要RPC才能获取所有生产商，显然是很耗时的，可以在validator中调用，然后validator内部共享的同时，caller可以利用闭包拿到结果，用于后续的业务逻辑。

Closure<List<String>> closure = new ClosureHandler<List<String>>() {

    private List<String> allManufacturers;

    @Override
    public List<String> getResult() {
        return allManufacturers;
    }

    @Override
    public void doExecute(Object... input) {
        allManufacturers = manufacturerService.getAllManufacturers();
    }
};

Result ret = FluentValidator.checkAll()
                            .putClosure2Context("manufacturerClosure", closure)
                            .on(car, validator)
                            .doValidate().result(toSimple());


public class CarValidator extends ValidatorHandler<Car> implements Validator<Car> {

    @Override
    public boolean validate(ValidatorContext context, Car car) {
        Closure<List<String>> closure = context.getClosure("manufacturerClosure");
        List<String> manufacturers = closure.executeAndGetResult();

        if (!manufacturers.contains(car.getManufacturer())) {
            context.addErrorMsg(String.format(CarError.MANUFACTURER_ERROR.msg(), car.getManufacturer()));
            return false;
        }

        return true;
    }
}

7 高级玩法
7.1 与JSR303规范最佳实现Hibernate Validator集成

Hibernate Validator是JSR 303 – Bean Validation规范的一个最佳的实现类库，他仅仅是jboss家族的一员，和大名鼎鼎的Hibernate ORM是系出同门，属于远房亲戚关系。很多框架都会天然集成这个优秀类库，例如Spring MVC的@Valid注解可以为Controller方法上的参数做校验。

FluentValidator当然不会重复早轮子，这么好的类库，一定要使用站在巨人肩膀上的策略，将它集成进来。

想要了解更多Hibernate Validator用法，参考这个链接。

下面的例子展示了@NotEmpty, @Pattern, @NotNull, @Size, @Length和@Valid这些注解装饰一个公司类（Company）以及其成员变量（Department）。

public class Company {
    @NotEmpty
    @Pattern(regexp = "[0-9a-zA-Z4e00-u9fa5]+")
    private String name;

    @NotNull(message = "The establishTime must not be null")
    private Date establishTime;

    @NotNull
    @Size(min = 0, max = 10)
    @Valid
    private List<Department> departmentList;

    // getter and setter...
}

public class Department {
    @NotNull
    private Integer id;

    @Length(max = 30)
    private String name;

    // getter and setter...
}

我们要在这个Company类上做校验，首先引入如下的jar包到pom.xml中。

<dependency>
    <groupId>com.baidu.unbiz</groupId>
    <artifactId>fluent-validator-jsr303</artifactId>
    <version>1.0.5</version>
</dependency>

默认依赖于Hibernate Validator 5.2.1.Final版本。

然后我们使用HibernateSupportedValidator这个验证器来在company上做校验，它和普通的Validator没任何区别。

Result ret = FluentValidator.checkAll()
                            .on(company, new HibernateSupportedValidator<Company>().setValidator(validator))
                            .on(company, new CompanyCustomValidator())
                            .doValidate().result(toSimple());
System.out.println(ret);

注意这里的HibernateSupportedValidator依赖于javax.validation.Validator的实现，也就是Hibernate Validator，否则它将不会正常工作。

下面是Hibernate Validator官方提供的初始化javax.validation.Validator实现的方法，供参考使用。

Locale.setDefault(Locale.ENGLISH); // speicify language
ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
javax.validation.Validator validator = factory.getValidator();

7.2 注解验证

一直以来介绍的方式都是通过显示的API调用来进行验证，FluentValidator同样提供简洁的基于注解配置的方式来达到同样的效果。

@FluentValidate可以装饰在属性上，内部接收一个Class[]数组参数，这些个classes必须是Validator的子类，这叫表明在某个属性上依次用这些Validator做验证。举例如下，我们改造下Car这个类：

public class Car {

    @FluentValidate({CarManufacturerValidator.class})
    private String manufacturer;

    @FluentValidate({CarLicensePlateValidator.class})
    private String licensePlate;

    @FluentValidate({CarSeatCountValidator.class})
    private int seatCount;

    // getter and setter ... 
}

然后还是利用on()或者onEach()方法来校验，这里只不过不用传入Validator或者ValidatorChain了。

Car car = getCar();

Result ret = FluentValidator.checkAll().configure(new SimpleRegistry())
                            .on(car)
                            .doValidate()
                            .result(toSimple());

那么问题来了？不要问我挖掘机（Validator）哪家强，先来说挖掘（Validator）从哪来。

默认的，FluentValidator使用SimpleRegistry，它会尝试从当前的class loader中调用Class.newInstance()方法来新建一个Validator，具体使用示例如下，默认的你不需要使用configure(new SimpleRegistry())。

一般情况下，SimpleRegistry都够用了。除非你的验证器需要Spring IoC容器管理的bean注入，那么你干脆可以把Validator也用Spring托管，使用@Service或者@Component注解在Validator类上就可以做到，如下所示：

@Component
public class CarSeatCountValidator extends ValidatorHandler<Integer> implements Validator<Integer> {
    // ...
}

这时候，需要使用SpringApplicationContextRegistry，它的作用就是去Spring的容器中寻找Validator。想要使用Spring增强功能，将下面的maven依赖加入到pom.xml中。

<dependency>
    <groupId>com.baidu.unbiz</groupId>
    <artifactId>fluent-validator-spring</artifactId>
    <version>1.0.5</version>
</dependency>

依赖树如下，请自觉排除或者更换不想要的版本。

[INFO] +- org.springframework:spring-context-support:jar:4.1.6.RELEASE:compile
[INFO] | +- org.springframework:spring-beans:jar:4.1.6.RELEASE:compile
[INFO] | +- org.springframework:spring-context:jar:4.1.6.RELEASE:compile
[INFO] | | +- org.springframework:spring-aop:jar:4.1.6.RELEASE:compile
[INFO] | | | - aopalliance:aopalliance:jar:1.0:compile
[INFO] | | - org.springframework:spring-expression:jar:4.1.6.RELEASE:compile
[INFO] | - org.springframework:spring-core:jar:4.1.6.RELEASE:compile
[INFO] +- org.slf4j:slf4j-api:jar:1.7.7:compile
[INFO] +- org.slf4j:slf4j-log4j12:jar:1.7.7:compile
[INFO] | - log4j:log4j:jar:1.2.17:compile

让Spring容器去管理SpringApplicationContextRegistry，由于这个类需要实现ApplicationContextAware接口，所以Spring会帮助我们把context注入到里面，我们也就可以随意找bean了。

调用的例子省略，原来怎么校验现在还怎么来。
7.3 分组

当使用注解验证时候，会遇到这样的情况，某些时候例如添加操作，我们会验证A/B/C三个属性，而修改操作，我们需要验证B/C/D/E 4个属性，显示调用FluentValidator API的情况下，我们可以做到“想验证什么，就验证什么”。

@FluentValidate注解另外一个接受的参数是groups，里面也是Class[]数组，只不过这个Class可以是开发人员随意写的一个简单的类，不含有任何属性方法都可以，例如：

public class Add {
}

public class Car {

    @FluentValidate(value = {CarManufacturerValidator.class}, groups = {Add.class})
    private String manufacturer;

}

那么验证的时候，只需要在checkAll()方法中传入想要验证的group，就只会做选择性的分组验证，例如下面例子，只有生产商会被验证。

Result ret = FluentValidator.checkAll(new Class<?>[] {Add.class})
                            .on(car)
                            .doValidate()
                            .result(toSimple());

这种groups的方法同样适用于hibernate validator，想了解更多请参考官方文档。
7.4 级联对象图

级联验证（cascade validation），也叫做对象图（object graphs），指一个类嵌套另外一个类的时候做的验证。

如下例所示，我们在车库（Garage）类中含有一个汽车列表（carList），可以在这个汽车列表属性上使用@FluentValid注解，表示需要级联到内部Car做onEach验证。

public class Garage {

    @FluentValidate({CarNotExceedLimitValidator.class})
    @FluentValid
    private List<Car> carList;
}

注意，@FluentValid和@FluentValidate两个注解不互相冲突，如下所示，调用链会先验证carList上的CarNotExceedLimitValidator，然后再遍历carList，对每个car做内部的生产商、座椅数、牌照验证。
7.5 Spring AOP集成

读到这里，你会发现，只介绍了优雅，说好的业务逻辑和验证逻辑解耦合呢？这需要借助Spring AOP技术实现，如下示例，addCar()方法内部上来就是真正的核心业务逻辑，注意参数上的car前面装饰有@FluentValid注解，这表示需要Spring利用切面拦截方法，对参数利用FluentValidator做校验。

@Service
public class CarServiceImpl implements CarService {

    @Override
    public Car addCar(@FluentValid Car car) {
        System.out.println("Come on! " + car);
        return car;
    }
}

Spring XML配置如下，首先定义了ValidateCallback表示该如何处理成功，失败，抛出不可控异常三种情况，因此，即使你拿不到Result结果，也可以做自己的操作，例如下面例子，如果发生错误，抛出CarException，并且消息是第一条错误信息，如果成功则打印"Everything works fine"，如果失败，把实际的异常e放入运行时的CarException抛出。

public class ValidateCarCallback extends DefaulValidateCallback implements ValidateCallback {

    @Override
    public void onFail(ValidatorElementList validatorElementList, List<ValidationError> errors) {
        throw new CarException(errors.get(0).getErrorMsg());
    }

    @Override
    public void onSuccess(ValidatorElementList validatorElementList) {
        System.out.println("Everything works fine!");
    }

    @Override
    public void onUncaughtException(Validator validator, Exception e, Object target) throws Exception {
        throw new CarException(e);
    }
}

将ValidateCarCallback注入到FluentValidateInterceptor中。然后利用BeanNameAutoProxyCreator在*ServiceImpl上用fluentValidateInterceptor拦截做验证，这有点类似Spring事务处理拦截器的使用方式。更多AOP的用法请参考Spring官方文档。

<bean id="validateCarCallback" class="com.baidu.unbiz.fluentvalidator.interceptor.ValidateCarCallback"/>

<bean id="fluentValidateInterceptor"
 class="com.baidu.unbiz.fluentvalidator.interceptor.FluentValidateInterceptor">
    <property name="callback" ref="validateCarCallback"/>
    <property name="locale" value="zh_CN"/>
    <property name="hibernateDefaultErrorCode" value="10000"/>
</bean>

<bean class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">
    <property name="beanNames">
        <list>
            <value>*ServiceImpl</value>
        </list>
    </property>
    <property name="interceptorNames">
        <list>
            <value>fluentValidateInterceptor</value>
        </list>
    </property>
</bean>

测试代码如下：

@Test
public void testAddCarNegative() {
    try {
        Car car = getValidCar();
        car.setLicensePlate("BEIJING123");
        carService.addCar(car);
    } catch (CarException e) {
        assertThat(e.getMessage(), Matchers.is("License is not valid, invalid value=BEIJING123"));
        return;
    }
    fail();
}

7.6 国际化支持

也就是常说的i18n，使用Spring提供的MessageSource来支持，只需要再Spring的XML配置中加入如下配置：

<bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
    <property name="basenames">
    <list>
        <value>error-message</value>
    </list>
    </property>
</bean>

同时编辑两个国际化文件，路径放在classpath中即可，通常放到resource下面，文件名分别叫做error-message.properties和error-message_zh_CN.properties表示默认（英文）和中文-简体中文消息。

car.size.exceed=Car number exceeds limit, max available num is {0}

car.size.exceed=u6c7du8f66u6570u91cfu8d85u8fc7u9650u5236uff0cu6700u591au5141u8bb8{0}u4e2a

注意，其中{0}代表和运行时填充的参数。

在Validator使用时，示例如下，只用MessageSupport.getText(..)来获取国际化信息。

public class GarageCarNotExceedLimitValidator extends ValidatorHandler&lt;Garage&gt; implements Validator&lt;Garage&gt; {

 public static final int MAX_CAR_NUM = 50;

 @Override
 public boolean validate(ValidatorContext context, Garage t) {
    if (!CollectionUtil.isEmpty(t.getCarList())) {
        if (t.getCarList().size() &gt; MAX_CAR_NUM) {
            context.addError(
 ValidationError.create(MessageSupport.getText(GarageError.CAR_NUM_EXCEED_LIMIT.msg(),
            MAX_CAR_NUM))
           .setErrorCode(GarageError.CAR_NUM_EXCEED_LIMIT.code())
           .setField(&quot;garage.cars&quot;)
           .setInvalidValue(t.getCarList().size()));
        return false;
        }
    }
    return true;
 }

}

这里需要注意下，如果使用HibernateValidator打的注解，例如@NotNull，@Length等，需要将错误信息放到ValidationMessages.properties和ValidationMessages_zh_CN.properties中，否则找不到哦。

8 总结

上面对FluentValidator做了一个全面的介绍，从显示的流式风格（Fluent Interface）API调用，以及各种丰富多样的链操作方法，再到对JSR303 – Bean Validation规范的集成，最后介绍了高级点的注解方式验证、支持级联对象图和Spring AOP的集成，我想这完成了对FluentValidator任务的诠释，更好的帮助开发人员做业务逻辑验证，希望能给你的项目一点帮助，哪怕不是用这个框架，而只是汲取一些思想，然后产生自己的思考，让我们的clean code更加“环保”。

如果有任何疑问，欢迎致电骚扰这个对代码有终身质保承诺的我。

更多实例，请参考github。