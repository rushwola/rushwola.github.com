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
