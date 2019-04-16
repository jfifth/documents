操作系统平台：Linux

打开oracle官网下载地址
![](assets\20190416075059.png)

使用wget下载
 wget  https://download.oracle.com/otn-pub/java/jdk/8u201-b09/42970487e3af4f5aa5bca3f542482c60/jdk-8u201-linux-x64.tar.gz

解压即可
tar  zxvf  jdk-8u201-linux-x64.tar.gz

目录结构如下
![](assets\20190416080011.png)

配置环境变量

```
vim  ~/.bash_profile
```

```
export  JAVA_HOME=/usr/java/jdk1.8.0_201
export  PATH=$PATH:$JAVA_HOME/bin
```

使配置生效

```
source  ~/.bash_profile
```

