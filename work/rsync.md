##使用rsync做多机文件同步
---

最近在思考项目中一个多机文件同步的方案，简单来说，有一台智能设备作为文件服务器，其他多台智能设备或PC作为客户端，将指定文件同步到服务器，并且支持从服务器恢复到客户端本地。有一定性能要求，希望能做到智能增量同步，最好还能保留同步的记录可以回溯。

时间紧，不想自己炮制算法，在网上找轮子。不知道是关键字搜索不得法还是没找到这方面高手聚集的地方，一直没能找到很对口的解决方案，直到有天无意中发现了[rsync](http://rsync.samba.org)这个东东。

这东东是*nix下多年以来著名的远程文件同步工具，用途很广，GPL，这里不展开描述了，详见官网，说说看中它的几个小理由：

- 支持远程文件同步（废话）
- 同步算法较为精良，业界良心。
- 支持ssh传输，安全性有保障
- 稳定成熟，开源。
- 轻量级，易移植、部署，使用方便。

今天找出两小时，在两台RHL服务器和一台Mac之间测试了一下，两台RHL服务器在异地的内网，在异地WAN路由器上做了SSH端口重定向，Mac是本机。

 - rsync服务器：10.0.1.21（RHL服务器）
 - rsync客户机A：10.0.2.2（RHL服务器）
 - rsync客户机B：192.168.1.100（Mac）

两台Linux一台BSD，Linux的rsync版本为3.x.x，BSD的rsync版本为2.6.9。测试即覆盖了局域网-广域网、又覆盖了异系统异rsync版本，还算全面:)

首先配置rsync服务器，先创建/work/tgt作为同步目录，然后在/etc下创建rsyncd.conf配置文件：
```
uid=root
gid=root
max connections = 2000
use chroot=no
log file=/var/log/rsyncd.log
pid file=/var/run/rsyncd.pid
lock file=/var/run/rsyncd.lock

[info]
path=/work/tgt
comment=sync infomation
ignore errors
read only=no
```
配置文件比较简单，里面配置了一个名为“info”的同步对象，物理路径是/work/tgt，后续在rsync客户机侧就可以用通过info这个名称来同步服务器上的文件。

再来说客户机的配置，有两种操作：同步数据到服务器；从服务器同步数据到本机。

####同步数据到服务器
```
rsync -avz -e ssh /work/src root@10.0.1.21:/work/tgt
rsync -avz -e 'ssh -p 12122' /work/src root@116.227.117.2:/work/tgt
```
第一条是LAN内另一台Linux的rsync客户机的同步操作，将本机/work/src目录同步到服务器的/work/tgt目录；第二条是远端Mac通过WAN路由器端口12122映射过去的同步操作。这里都使用了ssh作为传输通道，确保了数据同步的安全性。

####从服务器同步数据到本机
```
rsync -avz -e ssh 10.0.1.21::info /work/src
```
将服务器上info对象（对应物理目录/work/tgt）同步到本机/work/src目录。

尝试了多次，几个小文件间同步效果还不错，大规模文件同步有待考验性能。

这里有个小问题，通过ssh同步，每次都要输入ssh密码，[这里](http://aotee.com/ssh-rsync-files-without-password)介绍了一种无需每次输入密码的方法。

基本够用，剩下的遗留问题：
- 需要找一个iOS上的rsync移植方案，iOS没法像Android那样把rsync放在/bin中直接调用，只能看看能否找到rsync算法的lib方案在app中调用。
- rsync没有解决一个关键的同步历史记录问题，无法回溯。难道非得要用git包装一下才能完美解决？

以上两点如果没有解决方案，rsync还是无法作为项目的首选，有待进一步研究。

> 9/5/2013 Written with [StackEdit](http://benweet.github.io/stackedit/).