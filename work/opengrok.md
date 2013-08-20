## OpenGrok做WEB代码浏览
***
最近公司有新人报道，大伙在考虑ACP核心的平台部分代码如何逐步和受限的开放给新人，一方面考虑到新人不稳定期带来的信息泄露安全风险，一方面考虑到尽量提高新人学习的效率而不是天天对着代码黑盒乱猜。

今天在OS X上找类似Source Insight的APP时，无意中发现了lxr这个基于web的代码解析浏览工具，lxr望而生畏的复杂配置让各路人马都望洋兴叹，还好有[opengrok](https://github.com/OpenGrok/OpenGrok)这个简单易用的替代品。

一时间想到，能否把ACP的核心平台代码用opengrok部署起来，让新人通过web来浏览平台代码，虽然web代码也可以copy到本地保存，但至少比通过svn直接取到完整代码要费劲多了，从阅读的效率来说，opengrok提供的浏览功能包括了函数/方法跳转、关键字搜索、类/方法导航等操作，也基本够用了。

说干就干，跑到阿里云上搭建opengrok环境，大致流程如下：

* Java环境准备
* TOMCAT安装部署
* ctags安装
* opengrok[下载](http://java.net/projects/opengrok/downloads/download/opengrok-0.11.1.tar.gz)，解压直接使用
* 参照[帮助](https://github.com/OpenGrok/OpenGrok/wiki/How-to-install-OpenGrok)配置opengrok，将opengrok作为webapp部署到TOMCAT
* 配置opengrok浏览的源码目录，建立索引

比较简单，完成后，通过浏览器访问http://:8080/source就可以访问opengrok部署好的代码浏览界面了。

用了一会儿，发现一些问题：

* ctags对python的解析支持的不够理想，部分tag找不到，跳转失败
* 浏览界面上有Download功能，可以将当前浏览的代码文件下载到本地（功能本身是好的，但对我们不太合适，可以找机会修改opengrok的源码去掉它）
* 生成的代码浏览器页面不支持GBK编码的中文，都是乱码

这些问题感觉都比较要命，暂时只能放弃用opengrok作为新员工浏览学习关键代码的工具了。

小半天时间也不算白花，opengrok还是个好东西，找个机会研究一下lxr，看看能否有惊喜。

