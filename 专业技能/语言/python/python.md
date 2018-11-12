
#python

# 安装

到官网下载安装程序:https://www.python.org/downloads/

https://www.cnblogs.com/feng18/p/5854912.html

# IDE

PyCharm

# Python语法学习

学习网站：

https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000


# 安装jupyter notebook

```

#安装
pip install jupyter

#运行  默认访问路径http://localhost:8888
jupyter notebook

#指定访问端口
jupyter notebook --port 9999

```
##修改supyter notebook密码

首先，连接服务器，执行
jupyter notebook --generate -config 来生成配置文件
随后执行jupyter notebook password
输入密码即可

#安装statsmodels模块

到一个第三方网站：http://www.lfd.uci.edu/~gohlke/pythonlibs/#statsmodels，下载合适版本的whl文件
pip
 install 文件名.whl
搞定。如果没有wheel，先运行

pip
 install wheel

#Anaconda开发环境

##序
Python易用，但用好却不易，其中比较头疼的就是包管理和Python不同版本的问题，特别是当你使用Windows的时候。为了解决这些问题，有不少发行版的Python，比如WinPython、Anaconda等，这些发行版将python和许多常用的package打包，方便pythoners直接使用，此外，还有virtualenv、pyenv等工具管理虚拟环境。

个人尝试了很多类似的发行版，最终选择了Anaconda，因为其强大而方便的包管理与环境管理的功能。该文主要介绍下Anaconda，对Anaconda的理解，并简要总结下相关的操作

##Anaconda概述
Anaconda是一个用于科学计算的Python发行版，支持 Linux, Mac, Windows系统，提供了包管理与环境管理的功能，可以很方便地解决多版本python并存、切换以及各种第三方包安装问题。Anaconda利用工具/命令conda来进行package和environment的管理，并且已经包含了Python和相关的配套工具。

这里先解释下conda、anaconda这些概念的差别。conda可以理解为一个工具，也是一个可执行命令，其核心功能是包管理与环境管理。包管理与pip的使用类似，环境管理则允许用户方便地安装不同版本的python并可以快速切换。Anaconda则是一个打包的集合，里面预装好了conda、某个版本的python、众多packages、科学计算工具等等，所以也称为Python的一种发行版。其实还有Miniconda，顾名思义，它只包含最基本的内容——python与conda，以及相关的必须依赖项，对于空间要求严格的用户，Miniconda是一种选择。
进入下文之前，说明一下conda的设计理念——conda将几乎所有的工具、第三方包都当做package对待，甚至包括python和conda自身！因此，conda打破了包管理与环境管理的约束，能非常方便地安装各种版本python、各种package并方便地切换。

##功能介绍
Anaconda Navigtor ：用于管理工具包和环境的图形用户界面，后续涉及的众多管理命令也可以在 Navigator 中手工实现。
Jupyter notebook ：基于web的交互式计算环境，可以编辑易于人们阅读的文档，用于展示数据分析的过程。
qtconsole ：一个可执行 IPython 的仿终端图形界面程序，相比 Python Shell 界面，qtconsole 可以直接显示代码生成的图形，实现多行代码输入执行，以及内置许多有用的功能和函数。
spyder ：一个使用Python语言、跨平台的、科学运算集成开发环境。

# python人工智能

numpy, pandas,
scipy, sklearn,
tensorflow

最后一个是核武器
tensorflow
