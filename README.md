整这个起因是审项目的时候，遇到了可控的JDBC链接。但由于是高版本Mysql Connector/J（>=8.0.16），么的JDBC Attack提到的那个反序序列化。便想通过Rogue Mysql Server去读本地的jar包，以翻到数据库密码等敏感信息。但是Rogue Mysql Server不支持读取大文件，原因在：

![](https://github.com/xiaopan233/Rogue-MySql-Server/blob/master/1.png)

上图的代码中，是判断了`self.order`和Mysql数据包中的`packet_num`是否一致，不一致就报错。self.order是会一直自增的，但是Mysql数据包中的`packet_num`是从数据包中解析的，如下：

![](https://github.com/xiaopan233/Rogue-MySql-Server/blob/master/2.png)

Mysql数据包中的`packet_num`是切割数据包的第一个字节得到的。一个字节就是`255`，正常情况下，Rogue Mysql Server只能接受`255`个数据包，就会抛出错误。我这边测试发现，255个数据包大概是`17M`的样子。

可是我要读的jar包有100M哎，`17M`完全不够。

翻了下代码，发现`packet_num`除了校验数据包数量，似乎没什么用处，那我们干脆把关于`packet_num`报错的操作的注释了，不就可了？

然后就诞生了这个小脚本233

改脚本不仅支持读取无限量（理论上是这样）的文件，还可以读取的写入到指定的文件中。仅需要修改脚本中的`readJarFileName`为写入的目标文件即可。



使用效果，当有Mysql客户端连接上时：

![](https://github.com/xiaopan233/Rogue-MySql-Server/blob/master/3.png)

![](https://github.com/xiaopan233/Rogue-MySql-Server/blob/master/4.png)

最后会报下错，不过文件确实是读取完整了，能正常打开

![](https://github.com/xiaopan233/Rogue-MySql-Server/blob/master/5.png)
