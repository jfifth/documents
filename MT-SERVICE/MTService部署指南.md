# MT Service(MT Server)部署指南

Version 7.1.6

## 修订记录
<table>
  <thead>
    <tr>
      <th>版本</th>
      <th>修订时间</th>
      <th>修订内容</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>7.1.6</td>
      <td nowrap>2019-04-15</td>
      <td>部署指南初始化</td>
    </tr>
  </tbody>
</table>

## 摘要

MT（Memory Transfer） Service是用于数据快速交换的中间件服务，实现应用为mt server。

## 适用对象

接口开发人员和运维人员。

## 端口预设

| 端口  | 说明                |
| ----- | ------------------- |
| 11099 | RMI服务请求端口     |
| 11100 | RMI服务数据回传端口 |
| 18080 | HTTP服务端口        |

## 运行环境

- MT Server运行在JDK8环境上，RMI调用方需要使用JDK8及以上的环境；
- RMI调用时，需要把调用客户端放到本地工程classpath路径内，调用者就存在该jar包内，同时存在其他的相关辅助类；

## 安装

### 版本

mt-server-\*.\*.\*_hbase-\*.\*.\*.tar.gz

mt-server-(mt server版本号)_hbase-(hbase版本号).tar.gz

>  mt server适配不同的hadoop环境，为了兼容现场应用版本，加入了hbase版本支持信息。部署mt server时，请优先确定hbase的版本信息，选取对应的mt server版本。

### 环境预设

- 确定mt server绑定的各种服务IP和端口；
- 添加hosts对应关系并同hhbase环境保持统一；
- 关闭防火墙或者开放预设的端口；

### 安装

> 设定将要安装的mt server的版本为：mt-server-7.1.6_hbase-1.1.2.tar.gz

```
//解压安装包
> tar -xzvf mt-server-7.1.6_hbase-1.1.2.tar.gz
//解压后目录结构如下：
mt-server
	 // mt server启动和关闭及相关工具目录
     - bin
     	// 启动mt server
     	startup.sh
     	// 开始shell启动脚本的debug模式
     	startup4debug.sh
     	// 关闭mt server
     	shutdown.sh
     	// rmi 压力测试命令
     	rmi-stress-test.sh
     	// 工具脚本存放目录
     	tools
     	// 脚本环境配置信息
     	etc
     // 配置信息目录
     - conf
     	// mt server配置文件
     	application.yml
     	// 日志信息配置文档
     	log4j.properties
     	// 反馈信息文档，请勿修改
     	message
     	// 各种接口的Definition存放目录
     	tbls
     	// xdr规范信息存放目录
     	xdr
     // 日志文件目录
     - logs
     // 系统应用库
     - lib
     // 模板及静态文件存放目录
     - web
     // 调测及错误文件存放目录
     - raw
     // 压力测试结果及报告存放目录
     - stress-test
     // mt server应用数据存放目录
     - data
```

### 配置

mt server关键配置文件为：application.yml，配置文件采用yaml格式组织。

> yaml文件采用空格缩进，请勿采用其他缩进方式，否则会造成参数读取异常。

```
#mt-server configurations
mt-server:
	// 数据源类型，用于数据库模板表创建，值为：xdr,bins,all
    data-source-type: xdr
    // 接口查询超时阈值，单位为：秒
    query-timeout: 60
    // 异常字符串过滤列表
    exception-string-filtered: \N
    // es server列表信息
    elastic-server:
        hosts:
            - http://10.92.59.44:9200
            - http://10.92.59.43:9200
    // hbase 相关配置信息
    hbase:
    	// hbase 配置信息支持：file，ftp，http，sftp方式读取
        site-config-url: sftp://root:1233241@10.92.59.37:22//usr/hdp/2.6.5.0-292/hbase/conf/hbase-site.xml
        authentication-enable: true
        authentication-mechanism: kerberos
        authentication-kerberos:
            debug-enable: true
            principal: hbase@MH.COM
            keytab: /home/usr/hbase.keytab
    // bin文件解析相关配置参数
    bins:
    	// 是否开启小解码器的心跳服务
        mini-decoder-local-udpserver-enable: true
        // 小解码器的配置信息
        mini-decoder-local-udpserver:
            host: 1.1.1.1
            port: 2000
        // 读取bin文件规范的子目录
        tbl-files-relative-path: cuc-sichuan
        mix-data-from: signalingdata_1_{0},signalingdata_2_{0},signalingdata_3_{0}
        // 以上混合存放时，字段分隔符
        field-split-char: \\|
    xdr:
    	// xdr规范字段信息
        def-url: D:\\ise\\mt-server\\conf\\xdr\\cmcc-henan\\xdr-cmcc-henan.def
        // 以下对应码流获取的中间件服务相关信息
        xdrid-field-name: xdr_id
        xdrid-hex2long-enable: true
        xdr-field-default-separator: \|
        xdr-pcap-codestream-middleware:
            host: 134.235.202.29
            port: 1234
        xdr-pcap-export-type: file
        xdr-pcap-export-file-dir: /data/raw
        xdr-pcap-csfb-callid-fieldname: call_id
        xdr-pcap-csfb-ns: csfb
    // rmi调用参数
    rmi:
        host: 127.0.0.1
        port: 11099
        postback-port: 11100
        codestream-url-alias: /com/nsn/do/mt/hbase/xdr2bin4sichuan-cuc
        codestream-from-tbls: s1mme
    // http调用参数
    http:
       mt-url-alias: /com/nsn/do/mt/rest/cmcc-volte-beijing
       mtext-url-alias: /com/nsn/do/mt/hbase-es/xdr
       wait-timeout: 60
	// mt server应用上下午配置项
    context:
        concurrent-query-threshold: 100
        concurrent-export2db-threshold: 100
        concurrent-request-threshold: 100
        concurrent-handler4dispather-threshold: 100
        concurrent-data-processing-threshold: 100
        export:
            database:
                enable: false
                type: greenplum
                greenplum-schema: mt
                jdbc-url: jdbc:postgresql://192.168.3.82:5432/sqmmt
                user: mt
                password: mdasil
                jdbc-driver: org.postgresql.Driver
                jdbc-pool:
                    max-active: 100
                    min-idle: 50
                    initial-size: 50
                    validation-query: select 1
                    auto-commit: true
                error-debug-enable: true
                create-summary-table-function: mt_xdr_summary_create
                sqls-bulk-threshold: 10000
                timeout_threshold: 60

#spring-boot configurations
// 嵌入的spring boot配置信息
server:
    address: 127.0.0.1
    port: 18080
// 模板存放路径
spring:
    resources:
        static-locations:
            - file:${mt-server.base.dir}/web/static
    freemarker:
        suffix: .ftl
        template-loader-path:
            - file:${mt-server.base.dir}/web/templates
```

### 启动

```
// 切换到mt server/bin目录下
> cd bin
> startup.sh
```

### 核查是否启动成功

> 设定mt server绑定的http服务IP为：127.0.0.1，端口为：18080

1. 打开浏览器，输入：http://127.0.0.1:18080/version

2. 如果启动成功，则会输出类似如下内容：

   ```
   Version:7.1.6
   Builder:2019-04-14 07:26:59
   ```

   