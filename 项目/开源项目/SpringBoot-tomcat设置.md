# SpringBoot初始教程之Tomcat、Jetty优化以及Https配置

1.介绍

在SpringBoot的Web项目中，默认采用的是内置Tomcat，当然也可以配置支持内置的jetty，内置有什么好处呢？
1. 方便微服务部署。
2. 方便项目启动，不需要下载Tomcat或者Jetty

在目前的公司已经把内置的Jetty部署到了线上项目中，目前来说并无太大问题，内置就算有一些性能损失，但是通过部署多台机器，
其实也能够很轻松的解决这样的问题，内置容器之后其实是方便部署和迁移的。

1.1 优化策略

针对目前的容器优化，目前来说没有太多地方，需要考虑如下几个点

线程数
超时时间
jvm优化
针对上述的优化点来说，首先线程数是一个重点，初始线程数和最大线程数，初始线程数保障启动的时候，如果有大量用户访问，能够很稳定的接受请求，
而最大线程数量用来保证系统的稳定性，而超时时间用来保障连接数不容易被压垮，如果大批量的请求过来，延迟比较高，不容易把线程打满。这种情况在生产中是比较常见的
一旦网络不稳定，宁愿丢包也不愿意把机器压垮。
jvm优化一般来说没有太多场景，无非就是加大初始的堆，和最大限制堆,当然也不是无限增大，根据的情况进行调节

2. 快速开始

3.1 Tomcat SSL

tomcat的SSL配置很简单,先通过JDK的方式生成.keystore，这种方式的证书一般来说不太被认可的，最好的方式去网上申请，阿里云和腾讯云都可以免费申请，
这种方式配置出来的https，google浏览器会提示https不受认证

keytool -genkey -alias tomcat -keyalg RSA
application-tomcat.yaml

这块对tomcat进行了一个优化配置，最大线程数是100，初始化线程是20,超时时间是5000ms

server:
      tomcat:
        max-threads: 100
        min-spare-threads: 20
      connection-timeout: 5000
      ssl:
        key-store: classpath:.keystore
        key-store-type: JKS
        key-password: qq123456
        key-alias: tomcat
      port: 8443

      启动类

    启动类这块加上了一个httpConnector，为了支持https访问和http访问

    @SpringBootApplication
    public class AppApplication {
        public static void main(String args[]) {
            SpringApplication.run(AppApplication.class, args);
        }

        @Bean
        public EmbeddedServletContainerFactory tomcatEmbeddedServletContainerFactory() throws IOException {
            TomcatEmbeddedServletContainerFactory tomcat = new TomcatEmbeddedServletContainerFactory();
            tomcat.addAdditionalTomcatConnectors(httpConnector());
            return tomcat;
        }

        public Connector httpConnector() throws IOException {
            Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
            Http11NioProtocol http11NioProtocol = (Http11NioProtocol) connector.getProtocolHandler();
            connector.setPort(8080);
            //设置最大线程数
            http11NioProtocol.setMaxThreads(100);
            //设置初始线程数  最小空闲线程数
            http11NioProtocol.setMinSpareThreads(20);
            //设置超时
            http11NioProtocol.setConnectionTimeout(5000);
            return connector;
        }

    }

    上述就完成了https的配置，如果启动成功可以发现tomcat启动时候监听了两个端口.
    Tomcat initialized with port(s): 8443 (https) 8080 (http)。

    3.2 jvm优化

这块主要不是谈如何优化，jvm优化是一个需要场景化的，没有什么太多特定参数，一般来说在server端运行都会指定如下参数
初始内存和最大内存基本会设置成一样的，具体大小根据场景设置，我们线上环境一般都是4G，因为机器是16G的，-server是一个必须要用的参数，
至于收集器这些使用默认的就可以了，除非有特定需求

java -Xms4g -Xmx4g -Xmn768m -server -jar springboot-9-1.4.1.RELEASE.jar

4 jetty配置

pom.xml

springboot增加了一个starter针对jetty的，给pom增加一个依赖即可

<dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-jetty</artifactId>
       </dependency>

下面是针对jetty的线程优化，进行了一个配置，当然不配置也是可以使用的，在线上环境中最好配置一下，进行优
……
@Profile("jetty")
        @Bean
        public JettyEmbeddedServletContainerFactory jettyEmbeddedServletContainerFactory(
                JettyServerCustomizer jettyServerCustomizer) {
            JettyEmbeddedServletContainerFactory factory = new JettyEmbeddedServletContainerFactory();
            factory.addServerCustomizers(jettyServerCustomizer);
            return factory;
        }


        @Bean
        public JettyServerCustomizer jettyServerCustomizer() {
            return server -> {
                // Tweak the connection config used by Jetty to handle incoming HTTP
                // connections
                final QueuedThreadPool threadPool = server.getBean(QueuedThreadPool.class);
                threadPool.setMaxThreads(100);
                threadPool.setMinThreads(20);
            };
        }
……
4.1 jetty https配置

application-jetty.yaml

https配置和tomcat的没有太多差别，这块是统一配置，SpringBoot做了一个抽象化而已
server:
  connection-timeout: 5000
  ssl:
    key-store: classpath:.keystore
    key-store-type: JKS
    key-password: qq123456
    key-alias: tomcat
  port: 8444

  3 总结

一般来说在生产环境中不会用tomcat配置https，因为在我们的生产环境中，tomcat是一个统一的模板，只能够改线程数。一般的做法都是通过
nginx配置https，配置方式也比较简单，而且也方便重启
