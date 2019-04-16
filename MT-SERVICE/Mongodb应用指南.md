# Mongodb应用指南

Version 1.0.0

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
      <td>1.0.0</td>
      <td nowrap>2019-04-15</td>
      <td>应用指南初始版本</td>
    </tr>
  </tbody>
</table>
## 适用对象

- 应用开发
- 系统运维

## 安装

### 版本选择

> 当前应用mongodb的版本皆为社区版本，请通过官网选择对应的系统的编译版本；社区版本不支持ssl连接模式，如果需要ssl支持，需要通过源码重新编译开启该功能。

- 版本选择：3.6.12
- 针对Linux系统，版本区分为：
  - RHEL7（红帽系统7）：mongodb-linux-x86_64-**rhel70**-3.6.12.tgz
  - RHEL6（红帽系统6）：mongodb-linux-x86_64-**rhel62**-3.6.12.tgz
- 版本选择错误会造成mongodb的启动失败
- 具体官方安装文档参考地址：https://docs.mongodb.com/v3.6/administration/install-on-linux/

### 下载地址

https://www.mongodb.com/download-center/community

> 注意选取应用对应的系统版本

### 安装

> 建议mongodb的应用程序目录和数据目录不要设定在同一个存储硬盘上，数据处理的IO性能是mongodb的应用瓶颈

下载tgz文件解压：

​	tar -xzvf *******.tgz

解压后，可以使用./mongo --version测试安装是否成功。

## 配置

mongodb官方配置参数可以参考如下地址

https://docs.mongodb.com/v3.6/reference/configuration-options/

关键配置参数说明

```
#系统日志参数配置
systemLog:
	#日志输出类型
    destination: file
    #日志文件路径,需要绝对路径嘻嘻你
    path: "/opt/do/mzs/mongo/logs/standard/mongodb-000.log"
    #是否追加到以往的日志文件中,配置为false,防止日志文件过大影响mongodb的应用效率
    logAppend: false
#数据文件配置参数
storage:
	#数据文件目录及文件
    dbPath: "/opt/do/mzs/mongo/data/db000"
    wiredTiger:
        engineConfig:
        	#mongodb的内存占用阈值,请依据系统配置设定该参数,防止mongodb过多占用内存
            cacheSizeGB: 5
#网络参数
net:
	#绑定网络ID,建议同时绑定127.0.0.1地址作为本地管理的IP地址
    bindIp: 127.0.0.1,10.56.71.118
    #绑定的端口
    port: 30000   
processManagement:
	#是否daemon方式运行
    fork: true
#是否开启认证
security:
	#系统默认关闭认证功能（disabled），如果需要启用认证，需设定为：enabled
    authorization：enabled
```

> 需要把以上配置参数保存为yaml格式的配置文件，例如：mongodb-standard.conf作为启动参数文件；
>
> yaml文件格式使用空格缩进。

## 启动、访问、关闭

### 启动

> 设定配置参数文件为：mongodb-standard.conf

启动命令：mongod -f  mongodb-standard.conf &

启动后，会在日志文件中输出相关的启动信息，并启动mongod的进程信息，如果启动失败，会在日志文件中有所反馈。

> 如果mongodb未正常关闭，启动后会存在多个mongod的进程用于处理数据的恢复，该段时间内，mongodb并未开启对外提供服务，要避免mongodb启动后，马上启动相关应用的情况，需要等到mongodb稳定后在启动相关的应用。

### 访问

- 可以通过mongodb shell方式访问

  mongo --host=mongodb-ip --port=mongodb-port 

- 使用Robo客户端访问

  > Robo仅支持64位操作系统

  robo下载地址为：https://robomongo.org/download，选择Robo 3T版本

### 关闭

> 设定当前mongodb未开启认证，如果开启认证功能，需要使用--username和--password参数进行认证。设定mongodb的ip为127.0.0.1，端口为：30000	

```
./mongo --host=127.0.0.1 --port=30000
> use admin;
> db.shutdownServer();
> exit;
```

## 认证

### 注意事项

- mongodb默认不开启认证功能，如果需要开启认证，请配置authorization为：enabled
- 添加认证用户时，mongodb的安全配置authorization需预设为：disabled
- 用户添加后，需要对mongodb进行重启生效；

### 用户角色

- root，超级用户角色，谨慎赋权；
- readWriteAnyDatabase，读和写任意数据库；
- userAdminAnyDatabase，对任意数据库拥有管理员角色

### 添加用户

- 构造js脚本。用户添加js脚本，请复制以下脚本信息并修改mongodb的用户和密码，保存到对应的目录中，用于随后的mongodb的用户添加。

```
//mongodb server信息列表，多个server用逗号分隔
var mongodbs=[
               "127.0.0.1:30000"
             ];
             
function getLogDateTime(){
        return (new Date()).toString()+": ";
}
//公用的mongodb用户和密码
var mt_user="****";
var mt_pwd="********";

var mongo_user="*****";
var mongo_pwd="*********";

//添加mongodb的用户和角色
var add_users=[
    {
        user:mt_user,
        pwd:mt_pwd,
        roles:["userAdminAnyDatabase","dbAdminAnyDatabase","root"]
    },
    {
        user:mongo_user,
        pwd:mongo_pwd,
        roles:[
            "readWriteAnyDatabase",
            "userAdminAnyDatabase",
            "root"
        ]
    }
];

//执行添加
mongodbs.forEach(function(mongodb){
    print(getLogDateTime()+"Go to mongodb server:"+mongodb);
    //admin_db=connect(mongodb+"/admin",mt_user,mt_pwd);
    admin_db=connect(mongodb+"/admin");
    add_users.forEach(function(_user){
        try{
            print("Start dropping user:"+_user.user);
            admin_db.dropUser(_user.user);
        }catch(err){
            print("When dropping user:"+_user.user+" error");
            print(err+"\n");
        }
    });
    print("Start adding user...");
    add_users.forEach(function(_user){
        try{
            admin_db.createUser(
                {
                  user:_user.user,
                  pwd:_user.pwd,
                  roles:_user.roles
                }
            );
        }catch(err){
            print("An error has occurred:");
            print(err+"\n");
        }
    });
});
```

> 设定以上添加用户的js脚本保存位置为：/opt/do/js/add-user4mongodb.js

- 执行用户添加命令

  ./mongo  --nodb  /opt/do/js/add-user4mongodb.js,请注意执行该命令时，mongodb的认证功能为未开启状态

## 优化

### 存储

- Mongodb是NoSQL数据库类型，该数据库类型对内存和IO开销比较大，需要在硬件规划上给予考虑

### 系统参数

- 禁用numa

  numactl --interleave=all mongod .....

- Disable Transparent Huge Pages（禁用大页内存管理）

  - RHEL7

    ```
    #!/bin/bash
    
    zone_reclaim_mode=`cat /proc/sys/vm/zone_reclaim_mode`
    if  [ "$zone_reclaim_mode" == "1" ]; then
    	echo "Change /proc/sys/vm/zone_reclaim_mode to 0"
    	echo 0 > /proc/sys/vm/zone_reclaim_mode
    	sysctl -w vm.zone_reclaim_mode=0
    fi
    
    if [ -d /sys/kernel/mm/transparent_hugepage ]; then
      thp_path=/sys/kernel/mm/transparent_hugepage
    elif [ -d /sys/kernel/mm/redhat_transparent_hugepage ]; then
      thp_path=/sys/kernel/mm/redhat_transparent_hugepage
    else
      return 0
    fi
    
    echo 'never' > ${thp_path}/enabled
    echo 'never' > ${thp_path}/defrag
    
    echo 0  > ${thp_path}/khugepaged/defrag
    
    unset re
    unset thp_path
    ```

  - RHEL6

    ```
    #!/bin/bash
    
    zone_reclaim_mode=`cat /proc/sys/vm/zone_reclaim_mode`
    if  [ "$zone_reclaim_mode" == "1" ]; then
    	echo "Change /proc/sys/vm/zone_reclaim_mode to 0"
    	echo 0 > /proc/sys/vm/zone_reclaim_mode
    	sysctl -w vm.zone_reclaim_mode=0
    fi
    
    if [ -d /sys/kernel/mm/transparent_hugepage ]; then
      thp_path=/sys/kernel/mm/transparent_hugepage
    elif [ -d /sys/kernel/mm/redhat_transparent_hugepage ]; then
      thp_path=/sys/kernel/mm/redhat_transparent_hugepage
    else
      return 0
    fi
    
    echo 'never' > ${thp_path}/enabled
    echo 'never' > ${thp_path}/defrag
    
    echo 'no' > ${thp_path}/khugepaged/defrag
    
    unset re
    unset thp_path
    ```

- 关闭数据目录的noatime功能

  > 设定数据目录为：/data/mongodb

  chattr -R +A /data/mongodb

- 修改系统参数
  - ulimit -n 64000
  - ulimit -u unlimited