# Config Center

​	Config Center(配置中心) 是为各个模块提供集中配置文件管理的服务模块, 系统其它各个模块均可以通过HTTP方式从该服务提取配置信息, 这样做的好处是:

- 提供统一的配置入口
- 简化配置文件修改过程
- 共享通用信息,比如Tabledefinition
- 因配置文件导致的故障排查简单

#### 软件依赖

　　该模块赖如下软件:

- Linux环境,推荐RHEL6以上版本
- SUN JDK1.7+



#### 安装说明

　　该模块安装过程如下:

- 获取ConfiguratorXX.XX.XX.tar.gz并复制到服务器的/opt目录下.
- 执行命令:tar -xzf ConfiguratorXX.XX.XX.tar.gz
- 如果安装路径不是/opt, 那么需要修改ConfiguratorXX.XX.XX/run_config.sh 文件中的FRAMEWORK_HOME环境变量以匹配相应的安装路径
- 修改ConfiguratorXX.XX.XX/run_config.sh 文件中的JAVA_HOME环境变量,该变量应指向实际环境中的JDK主目录.



#### 配置选项

该系统自身只有1个配置文件:web.properties, 该文件提供对配置中心服务端口的配置. 具体如下.

##### web.properties

| 配置项   | 类型 | 默认值 | 说明                     |
| -------- | ---- | ------ | ------------------------ |
| web.port | 整数 | 8899   | 配置该服务的HTTP监听端口 |

#### 接口说明

　　其他模块可以通过地址 `http://配置中心IP:配置中心端口号/c/c?s=实例ID&r=实例类型/配置文件名` 获取配置文件内容.

- 配置中心IP: 配置中心服务器的IP地址
- 配置中心端口号: 配置中心HTTP服务监听端口号, 默认为:8899
- 实例ID: 为区分相同程序的多个不同实例, 要求每个实例有一个唯一的ID, 获取配置信息是此ID会作为获取配置文件的重要参考信息
- 实例类型: 一般是程序的名称, 比如:KpiAnalyzer, DataLoader, MongoImporter等
- 配置文件名: 要获取的配置文件名称

　　例如:http://localhost:8899/c/c?s=1&r=KpiAnalyzer/business.properties



#### 维护说明

　　通常配置文件只有一个,我们只需要修改这个文件即可,但是在集群环境下,同样的程序会部署多套：比如KpiAnalyzer在做集群时会部署多个KpiAnalyzser实例,而每个实例所用的配置文件内容不尽相同, 这就要求配置中心能够根据实例来发送不同的配置信息. 配置中心采用下面的方法流程来返回配置文件:

- 获取参数:实例ID,实例类型,配置文件名
- 在配置服务中心的configurator目录中寻找文件: '实例类型/配置文件名[实例ID]' , 如果找到文件,则将此文件发送给获取者,如果没有找到,则执行下面步骤
- 在配置服务中心的configurator目录中寻找文件: '实例类型/配置文件名' , 如果找到文件,则将此文件发送给获取者,否则返回错误:http 404

　　从上面的流程分析,实例ID会被优先使用来查找配置信息, 因此,相同的程序,其实例ID不能相同. 那么假设我们有三个KpiAnalyzser, ID分别为1,2,3. 再假设服务器上存在如下文件:

- KpiAnalzyer/business.properties
- KpiAnalzyer/business.properties[1]
- KpiAnalzyer/business.properties[3]

　　我们知道KpiAnalyzer在启动时会加载business.properties文件, 那么针对上面的情况,其结果是:

- 服务器(ID=2) 获取到了 KpiAnalzyer/business.properties (因为不存在文件:business.properties[2])
- 服务器(ID=1) 获取到了 KpiAnalzyer/business.properties[1]
- 服务器(ID=3) 获取到了 KpiAnalzyer/business.properties[3]



　　根据以上信息, 我们可以很容易的在一个集中的配置环境中为分布式环境里的各个软件系统提供配置信息.

#### 启动和关闭

启动:
```shell
cd /opt/KpiAnalyzerXX.XX.XX
./startup.sh
```
关闭:
```shell
cd /opt/KpiAnalyzerXX.XX.XX
./shutdown.sh
```