# commons-cli使用介绍

网页资料：http://www.cnblogs.com/xing901022/p/5608823.html

http://www.cnblogs.com/xing901022/p/5612328.html

commons-cli是Apache开源组织提供的用于解析命令行参数的包，命令行的处理共分为三个阶段：定义阶段、解析阶段和审讯阶段。
在定义阶段，我们需要使用Options类来定义我们需要使用的命令。

Apache Commons CLI是开源的命令行解析工具，它可以帮助开发者快速构建启动命令，并且帮助你组织命令的参数、以及输出列表等。

CLI分为三个过程：

* 定义阶段：在Java代码中定义Optin参数，定义参数、是否需要输入值、简单的描述等

* 解析阶段：应用程序传入参数后，CLI进行解析

* 询问阶段：通过查询CommandLine询问进入到哪个程序分支中

举个栗子

定义阶段:

```
Options options = new Options();
Option opt = new Option("h", "help", false, "Print help");
opt.setRequired(false);
options.addOption(opt);

opt = new Option("c", "configFile", true, "Name server config properties file");
opt.setRequired(false);
options.addOption(opt);

opt = new Option("p", "printConfigItem", false, "Print all config item");
opt.setRequired(false);
options.addOption(opt);

```

其中Option的参数：

*  第一个参数：参数的简单形式

* 第二个参数：参数的复杂形式

* 第三个参数：是否需要额外的输入

* 第四个参数：对参数的描述信息

解析阶段
通过解析器解析参数

```
CommandLine commandLine = null;
CommandLineParser parser = new PosixParser();
try {
    commandLine = parser.parse(options, args);
}catch(Exception e){
    //TODO xxx
}

```

询问阶段
根据commandLine查询参数，提供服务

```

HelpFormatter hf = new HelpFormatter();
hf.setWidth(110);

if (commandLine.hasOption('h')) {
    // 打印使用帮助
    hf.printHelp("testApp", options, true);
}

```

全部代码样例

```

package hangout.study;

import org.apache.commons.cli.CommandLine;
import org.apache.commons.cli.CommandLineParser;
import org.apache.commons.cli.HelpFormatter;
import org.apache.commons.cli.Option;
import org.apache.commons.cli.Options;
import org.apache.commons.cli.ParseException;
import org.apache.commons.cli.PosixParser;

public class CLITest {
    public static void main(String[] args) {
        String[] arg = { "-h", "-c", "config.xml" };
        testOptions(arg);
    }
    public static void testOptions(String[] args) {
        Options options = new Options();
        Option opt = new Option("h", "help", false, "Print help");
        opt.setRequired(false);
        options.addOption(opt);

        opt = new Option("c", "configFile", true, "Name server config properties file");
        opt.setRequired(false);
        options.addOption(opt);

        opt = new Option("p", "printConfigItem", false, "Print all config item");
        opt.setRequired(false);
        options.addOption(opt);

        HelpFormatter hf = new HelpFormatter();
        hf.setWidth(110);
        CommandLine commandLine = null;
        CommandLineParser parser = new PosixParser();
        try {
            commandLine = parser.parse(options, args);
            if (commandLine.hasOption('h')) {
                // 打印使用帮助
                hf.printHelp("testApp", options, true);
            }

            // 打印opts的名称和值
            System.out.println("--------------------------------------");
            Option[] opts = commandLine.getOptions();
            if (opts != null) {
                for (Option opt1 : opts) {
                    String name = opt1.getLongOpt();
                    String value = commandLine.getOptionValue(name);
                    System.out.println(name + "=>" + value);
                }
            }
        }
        catch (ParseException e) {
            hf.printHelp("testApp", options, true);
        }
    }
}

```

运行结果：

```

usage: testApp [-c <arg>] [-h] [-p]
 -c,--configFile <arg>   Name server config properties file
 -h,--help               Print help
 -p,--printConfigItem    Print all config item
--------------------------------------
help=>null
configFile=>config.xml

```
