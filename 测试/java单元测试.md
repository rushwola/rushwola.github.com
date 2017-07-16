��������:

http://www.cnblogs.com/AloneSword/p/4109407.html

一篇文章(http://blog.csdn.net/liushuijinger/article/details/32140843):

单元测试之道Java版：使用JUnit——程序员修炼三部曲 第二部

测试在软件生命周期中的重要性，不用我多说想必大家也都非常清楚。软件测试有很多分类，从测试的方法上可分为：黑盒测试、白盒测试、静态测试、动态测试等；从软件开发的过程分为：单元测试、集成测试、确认测试、验收、回归等。


在众多的分类中，与开发人员关系最紧密的莫过于单元测试了。像其他种类的测试基本上都是由专门的测试人员来完成，只有单元测试是完全由开发人员来完成的。那么今天我们就来说说什么是单元测试，为什么要进行单元测试，以及如更好的何进行单元测试。


什么是单元测试？

单元测试（unit testing），是指对软件中的最小可测试单元进行检查和验证。比如我们可以测试一个类，或者一个类中的一个方法。


为什么要进行单元测试？

为什么要进行单元测试？说白了就是单元测试有什么好处，其实测试的好处无非就是减少bug、提高代码质量、使代码易于维护等。单元测试有什么好处请看一下百度百科中归纳的四条：


1、它是一种验证行为。
程序中的每一项功能都是测试来验证它的正确性。它为以后的开发提供支援。就算是开发后期，我们也可以轻松的增加功能或更改程序结构，而不用担心这个过程中会破坏重要的东西。而且它为代码的重构提供了保障。这样，我们就可以更自由的对程序进行改进。


2、它是一种设计行为。
编写单元测试将使我们从调用者观察、思考。特别是先写测试（test-first），迫使我们把程序设计成易于调用和可测试的，即迫使我们解除软件中的耦合。


3、它是一种编写文档的行为。
单元测试是一种无价的文档，它是展示函数或类如何使用的最佳文档。这份文档是可编译、可运行的，并且它保持最新，永远与代码同步。


4、它具有回归性。
自动化的单元测试避免了代码出现回归，编写完成之后，可以随时随地的快速运行测试。


如何更好的进行单元测试？


在讨论如何更好的进行单元测试之前，先来看看我们以前是怎么测试代码的。

以前是这样测试程序的：
public int add(int x,int y) {  
    return x + y;  
}  

public static void main(String args[]) {  
    int z = new Junit().add(2, 3);  
    System.out.println(z);  
}

如上面所示，在测试我们写好的一个方法时，通常是用一个main方法调用一下我们要测试的方法，然后将结果打印一下。现在看来这种方式已经非常out了，所以出现了很多单元测试的工具，如：JUnit、TestNG等。借助它们可以让我们的单元测试变得非常方便、高效。今天就说说如何利用JUnit进行单元测试。


我们新建一个Java Project以便进行演示，至于java Project怎么创建我就不在此赘述了，如果连怎么建Java Project，那你还不适合看这篇文章。建好以后在该项目的“src”目录上右击，选择new——》JUnit Test Case，然后按下图填写必要信息：



填写好包名和类名（选择New JUnit 4 Test），点击最下面的那个“Browse”按钮来选择需要测试的类：



手动输入我们要测试的类，选择该类，点击“OK”，回到第一张图的界面，然后点击“Next”，来到下图：



勾选要测试的方法，点击“Finish”，这样我们的JUnit测试实例就建好了。然后就可以写具体的测试了：

[java] view plain copy

    package com.tgb.junit.test;  

    //静态引入  
    import static org.junit.Assert.*;  
    import static org.hamcrest.Matchers.*;  

    import org.junit.Test;  

    import com.tgb.junit.Junit;  

    public class JUnitTest {  

        @Test  
        public void testAdd() {  
            int z = new  Junit().add(2, 3);  
            assertThat(z , is(5));  
        }  

        @Test  
        public void testDivide() {  
            int z = new Junit().divide(4, 2);
            assertThat(z, is(2));  
        }  
    }  


写好以后，右击该类选择“Run As”——》“JUnit Test”，出现下图代表测试通过：



到这里，可能有人会有疑问，JUnit跟用main方法测试有什么区别呢？

首先，JUnit的结果更加直观，直接根据状态条的颜色即可判断测试是否通过，而用main方法你需要去检查他的输出结果，然后跟自己的期望结果进行对比，才能知道是否测试通过。有一句话能够很直观的说明这一点——keeps the bar green to keeps the code clean。意思就是说，只要状态条是绿色的，那么你的代码就是正确的。

第二点，JUnit让我们同时运行多个测试变得非常方便，下面就演示一下如何进行多实例测试：

首先我们要再建一个待测试类，然后再建一个对应的JUnit测试实例，步骤略。然后在我们测试实例的包上右击选择“Run As”——》“Run Configurations”，如下图；



选择第二项“Run all tests in the selected project, package or source folder”，然后点击“Run”效果如下：



可以看到，我们本次测试了两个类，共三个方法，这种方便的效果在测试实例越多的情况下，体现的越明显。至于main方法运行多个测试，想想就觉得非常麻烦，这里就不演示了。


JUnit除了可以测试这些简单的小程序，还可以测试Struts、JDBC等等，这里只是用这个小程序做过简单的介绍。本实例使用的是hamcrest断言，而没有使用老的断言，因为hamcrest断言更加接近自然语言的表达方式，更易于理解。


本实例需要引入以下三个jar包：

hamcrest-core-1.3.jar
hamcrest-library-1.3.jar
junit-4.10.jar


最后附上常用hamcrest断言的使用说明：

[java] view plain copy

    数值类型  
    //n大于1并且小于15，则测试通过  
    assertThat( n, allOf( greaterThan(1), lessThan(15) ) );  
    //n大于16或小于8，则测试通过  
    assertThat( n, anyOf( greaterThan(16), lessThan(8) ) );  
    //n为任何值，都测试通过  
    assertThat( n, anything() );  
    //d与3.0的差在±0.3之间，则测试通过  
    assertThat( d, closeTo( 3.0, 0.3 ) );  
    //d大于等于5.0，则测试通过  
    assertThat( d, greaterThanOrEqualTo (5.0) );  
    //d小于等于16.0，则测试通过  
    assertThat( d, lessThanOrEqualTo (16.0) );  

    字符类型  
    //str的值为“tgb”，则测试通过  
    assertThat( str, is( "tgb" ) );  
    //str的值不是“tgb”，则测试通过  
    assertThat( str, not( "tgb" ) );  
    //str的值包含“tgb”，则测试通过  
    assertThat( str, containsString( "tgb" ) );  
    //str以“tgb”结尾，则测试通过  
    assertThat( str, endsWith("tgb" ) );
    //str以“tgb”开头，则测试通过  
    assertThat( str, startsWith( "tgb" ) );
    //str忽略大小写后，值为“tgb”，则测试通过  
    assertThat( str, equalToIgnoringCase( "tgb" ) );
    //str忽略空格后，值为“tgb”，则测试通过  
    assertThat( str, equalToIgnoringWhiteSpace( "tgb" ) );  
    //n与nExpected相等，则测试通过（对象之间）  
    assertThat( n, equalTo( nExpected ) );

    collection类型  
    //map中包含key和value为“tgb”的键值对，则测试通过  
    assertThat( map, hasEntry( "tgb", "tgb" ) );  
    //list中包含“tgb”元素，则测试通过  
    assertThat( iterable, hasItem ( "tgb" ) );  
    //map中包含key为“tgb”的元素，则测试通过  
    assertThat( map, hasKey ( "tgb" ) );  
    //map中包含value为“tgb”的元素，则测试通过  
    assertThat( map, hasValue ( "tgb" ) );  
