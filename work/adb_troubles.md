
Mac OS X adb调试的常见问题
=====================
----------
前阵子搞好了OS X下识别Android手机和adb调试，遇到了一些问题，花了不少时间，总结如下。

问题1：手机插入后无法识别
---------
在Windows下一般是安装ADB驱动，在OS X下驱动层面已经OK，所需要的只是让adb能正确识别该手机的USB vendor ID即可。
插入手机后，点击Apple->关于本机->更多信息->系统报告->硬件->USB，应该能找到插入的手机设备，查看信息如下：
>  产品 ID：	0x5173
>
>  **厂商 ID：	0x0fce  (Sony Ericsson Mobile Communications AB)**
>
>  版本：	 2.16
>
>  序列号：	BX90382Z7V
>
>  速度：	最大 480 Mb/秒
>
>  制造商：	ST-Ericsson
>
>  位置 ID：	0x1d110000 / 6
>
>  可用电流 (mA)：	500
>
>  所需电流 (mA)：	500

注意其中**厂商ID**即可，将这个ID加入到~/.android/adb_usb.ini文件中。
adb即可识别这个设备。

问题2：adb启动server报错didn't ack...
---------
这个问题比较麻烦，出现的莫名其妙，也花了不少时间才搞定。
总的来说，有如下原因可能导致该问题发生：

- 已有adb进程存在
- 其他进程占用5037端口
- adb_usb.ini文件格式不合法

在网上搜索解决方案时，大部分帖子指出的都是前两个原因，这两个原因很好解决，netstat查看一下，找到已经开启的adb或者占用5037端口的进程杀掉即可。

第三个原因是在无意中执行adb nodaemon模式时才发现的：
>allen-mac-mini:platform-tools xuweinan$ ./adb nodaemon server
>
>Invalid content in adb_usb.ini. Quitting.

回过头去看adb_usb.ini，发现多了一个空行，删除掉，再重新启动adb server，就一切正常了。


Written with [StackEdit](https://stackedit.io/).
