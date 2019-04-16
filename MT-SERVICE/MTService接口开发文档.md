# MT Service接口开发文档

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
      <td nowrap>2019-03-28</td>
      <td>添加SQL解析错误提示功能，增添了返回代码说明，增添了double类型，增加了信令回溯接口，增添了返回记录数字段信息便于数据统计</td>
    </tr>
      <tr>
    <td>7.1.5</td>
    <td>2019-03-25</td>
    <td>添加了SQL翻译类型判断的功能，如果查询字段内容在接口表中不存在，直接忽略该查询条件，针对日期类型(datetime)的参数按照yyyy-MM-dd HH:mm:ss.SSS格式传入，屏蔽不同存储平台日期类型的差异，修改了HTTP调用类型为REST</td>
</tr>
<tr>
    <td>7.1.4</td>
    <td>2019-03-19</td>
    <td>实现小解码器Lte解码对接功能，增添规范字段tinyint，bigint，varchar的字段，主要针对bin文件解析字段类型</td>
</tr>
<tr>
    <td>7.1.3</td>
    <td>2019-03-14</td>
    <td>修改了终端型号匹配规则,增添了is null,is not null ,like ,not like叶子查询语句的支持,针对SQL解析重新实现</td>
</tr>
<tr>
    <td>7.1.2</td>
    <td>2019-03-01</td>
    <td>SQL查询接口发布</td>
</tr>
<tr>
    <td>7.1.1</td>
    <td>2019-02-18</td>
    <td>修改了数据中存在数组的解析规范,增添标准接口中imsis和msisdns多数据查询</td>
</tr>
<tr>
    <td>7.1.0</td>
    <td>2019-02-12</td><td>修改了返回数据header中添加mt_row_id的字段信息</td>
</tr>
<tr>
    <td>7.1.0</td>
    <td>2019-02-12</td><td>修改规范接口参数名称tblLimits----&gt;tblLimit</td>
</tr>
  </tbody>
</table>
## 摘要

MT（Memory Transfer） Service是用于数据快速交换的中间件服务，实现应用为mt server。

## 适用对象

接口开发人员，本文档不适用运维人员。

## 术语

| 名称 |               说明                    |
| ---- | --------------------------------- |
| 规范接口 | 请求参数固定的调用方式 |
| SQL查询接口 | 请求参数不固定的接口调用方式      |
| MT SQL | 针对MT Server适配的结构化查询(SQL方言 |
| 定值 | 即数据常量，值为不可变 |
| rest | http协议的RESTFul接口风格调用方式 |
| rmi  | RMI接口调用方式 |
| ICommonInvoker | RMI接口调用者 |
| mt-server-client-\*\*\*\*.jar | RMI接口调用客户端 |

## 端口预设

| 端口  | 说明                |
| ----- | ------------------- |
| 11099 | RMI服务请求端口     |
| 11100 | RMI服务数据回传端口 |
| 18080 | HTTP服务端口        |

## 运行环境

- MT Server运行在JDK8环境上，RMI调用方需要使用JDK8及以上的环境；
- RMI调用时，需要把调用客户端放到本地工程classpath路径内，调用者就存在该jar包内，同时存在其他的相关辅助类；

## 辅助URL

> 设定当前mt server的url为：http://localhost:18080

- mt server版本查看

  http://localhost:18080/version

- mt client下载与查看

  http://localhost:18080/mt-clients

- mt-spec规范地址

  http://localhost:18080/mt-spec

- mt-sql解析测试地址:

  http://localhost:18080/helper/mtsql-parser

- mt-server字段信息

  - xdr 字段信息

    http://localhost:18080/helper/xdr-fields

  - tblDefinition字段信息[内容比较多,打开后需要等待加载完成]

    http://localhost:18080/helper/bin-fields

## 返回状态码

| 代码 | 说明                 |
| ---- | -------------------- |
| 000  | 业务处理成功         |
| 200  | 参数验证失败         |
| 201  | MT SQL 解析失败      |
| 404  | 请求业务url不存在    |
| 500  | 业务处理出现内部错误 |

## MT SQL(SQL 方言)

### MT SQL 关键字

| 关键字      | 说明 [关键字不区分大小写]                                    |
| ----------- | ------------------------------------------------------------ |
| get         | 字段语句，当前统一设定为：get *                              |
| from        | 数据表字段信息，多个表字段使用逗号分隔                       |
| case        | 条件语句，需要和from中设定的表一一对应，并需要{}指定范围，多个条件语句使用逗号分隔 |
| limit       | 返回数据记录限制，使用方法为：limit from size，例如：limit 0 2，从下标0开始，返回2条数据记录 |
| is null     | 字段缺失、字段内容为空（字符串和field=‘’等效）               |
| is not null | 存在该字段内容、字段内容不为空（字符串和field!=‘’等效）      |
| like        | 模糊查询，性能比较低                                         |
| not like    | 模糊查询，性能比较低                                         |

### MT SQL语法规则

- 关键字和条件语句之间需要留出空格，否则无法识别；
- 查询表需要同case语句一一对应关系，否则查询无法提交；
- 查询字段类型
  - 字符串类型，请使用**单引号**包起，如果字符串中存在单引号，请使用**\\'**进行转义；
  - 数值类型，可以使用单引号包起，也可以直接使用明值；
  - 时间类型，请按照**yyyy-MM-dd HH:mm:ss.SSS**格式化，MT Server屏蔽各存储平台类型的日期类型的差异；
- 请注意 limit语句使用方法，开始记录下标和返回记录数中间请使用空格分开；

### 特殊查询约定

- 终端型号下钻使用字段：imei_sub，查询条件可以使用IN语句，例如：

  get * from cmcc_bigxdr_reg  case { imei_sub in ('86418403','86896502') }

### SQL查询

- get * from 表 ...... case {clauses limit 0 100 },{clauses limit 0 100 },{clauses limit 0 100 },{clauses limit 0 100 }......

  > limit语句， 可以依据需要设定为可选语句；
  >
  > 语句可以跨行；

#### 简单查询

- 等值查询[=]

  get * from cmcc_bigxdr_reg case  { imsi='460008626943542' }

- 不等查询[!=]

  get * from cmcc_bigxdr_reg,cmcc_bigxdr_call case  { imsi='460008626943542' },{ local_city !=' 376' }

- 大于查询（支持日期格式）[>]

  get * from cmcc_bigxdr_reg,cmcc_bigxdr_call case  { imsi='460008626943542' },{ local_city !=' 376' }

- 大于等于查询（支持日期格式）[>=]

  get * from cmcc_bigxdr_reg,cmcc_bigxdr_call case  { flow_endtime >='2019-01-28 15:23:26.467' },{ local_city !=' 376' }

- 小于查询（支持日期格式）[<]

  get * from cmcc_bigxdr_reg,cmcc_bigxdr_call case  { flow_endtime < '2019-01-28 15:23:26.467' },{ local_city !=' 376' }

- 小于等于查询（支持日期格式）[<=]

  get * from cmcc_bigxdr_reg,cmcc_bigxdr_call case  { flow_endtime <= '2019-01-28 15:23:26.467' },{ local_city !=' 376' }

- in查询[in]

  get * from cmcc_bigxdr_reg,cmcc_bigxdr_call case  { flow_endtime <= '2019-01-28 15:23:26.467' },{ local_city in ('376','2731','123') }

- not in查询[not in]

  get * from cmcc_bigxdr_reg,cmcc_bigxdr_call case  { flow_endtime <= '2019-01-28 15:23:26.467' },{ local_city not in ('376','2731','123') }

- is null

  get * from cmcc_bigxdr_reg,cmcc_bigxdr_call case  { flow_endtime  is null },{ local_city  is null }

- is not null

  get * from cmcc_bigxdr_reg,cmcc_bigxdr_call case  { flow_endtime  is not null },{ local_city  is  not  null }

- like 

  get * from cmcc_bigxdr_reg,cmcc_bigxdr_call case  { flow_endtime like 'A%B' },{ local_city like  'A%B'  }

- not like

  get * from cmcc_bigxdr_reg,cmcc_bigxdr_call case   { flow_endtime  not like 'A%B' },{ local_city not like  'A%B'  }

#### 并(and)查询

- get * from  cmcc_bigxdr_reg case { flow_endtime <= '2019-01-28 15:23:26.467' and local_city not in ('376','2731','123') and  imsi='460008626943542' }

#### 或(or)查询

- get * from  cmcc_bigxdr_reg case { flow_endtime <= '2019-01-28 15:23:26.467' or local_city not in ('376','2731','123')  or  imsi='460008626943542' }

#### 复合查询

- get * from cmcc_bigxdr_reg case { ( flow_endtime <= '2019-01-28 15:23:26.467' and local_city not in ('376','2731','123') and  imsi='460008626943542') and (flow_endtime <= '2019-01-28 15:23:26.467' or local_city not in ('376','2731','123')  or  imsi='460008626943542')}
- get * from cmcc_bigxdr_reg case { ( flow_endtime <= '2019-01-28 15:23:26.467' and local_city not in ('376','2731','123') and  imsi='460008626943542') or (flow_endtime <= '2019-01-28 15:23:26.467' or local_city not in ('376','2731','123')  or  imsi='460008626943542')}
- get *  from cmcc_bigxdr_call case  {   callfail_flag in ('WC10','WC70')  and ((call_side in ('1', '2') and s1uxdr_number>'0' and mwxdr_number>'0' and s1mmexdr_number>'0') or call_side='2') and flow_endtime >= '2019-01-28 15:00:00.000' and flow_endtime <= '2019-01-28 16:00:00.000' AND flow_firfail_netype='IS_CSCF' } 

#### Limit限制语句

- limit from size

  get * from cmcc_bigxdr_reg,cmcc_bigxdr_call case { (imsi='460008626943542' and local_city!='376') or ( msisdn='8618837508021' or imsi='460008626943542' ) limit 0 1 },{ msisdn='8618837508021' or imsi='460008626943542' limit 0 1}

## RMI

### 调用示例

```
import com.nsn.mt.rmi.ICommonInvoker;
import com.nsn.mt.rmi.MTServerInvokerHelper;
import com.nsn.mt.service.Request;
import com.nsn.mt.service.Response;

import java.net.MalformedURLException;
import java.rmi.NotBoundException;
import java.rmi.RemoteException;
import java.util.Map;

/**
 * @Auther: BISCUIT_0
 * @Date: 2019/03/19 16:23
 * @Description: RMI invoker demo
 */
public class RMIInvokerDemo {
    public static void main(String[] args) {
        Request request = new Request();
        Map<String, Object> param = request.getParameter();
        request.setUrl("/com/nsn/do/mt/hbase/xdr2bin4sichuan-cuc");
        param.put("imsi", "-1");
        param.put("msisdn", "8615680000000");
        param.put("startTime", "2019-03-19 10:40:00");
        param.put("endTime", "2019-03-19 10:41:59");
        param.put("tblLimit", "s1mme");
        request.setParameter(param);
        try {
            ICommonInvoker invoke = MTServerInvokerHelper.getRMIInvoker("192.168.17.225", "11099");
            Response response = invoke.query(request);
            System.out.println(response);
        } catch (RemoteException e) {
            e.printStackTrace();
        } catch (NotBoundException e) {
            e.printStackTrace();
        } catch (MalformedURLException e) {
            e.printStackTrace();
        }
    }
}
```

### 调用实体

#### Request

- String version; //版本信息，多版本情况下，通过版本号码分配调用方法
- String url;//资源定位符，例如：/com/nsn/do/mt/hbase/xdr，该地址需要向MT Server端获取
- String location;//针对应用地方进行参数指定
- Map<String,Object> paramMap=new HashMap<>();//具体请求地址同url相关联，不同的url使用的参数不同，key值需要固定
- String username;//登录的用户名
- String password;//登录的密码
- boolean cacheEnable =false;//是否开启Cache的功能

 #### Response

- public String getCode()； //本次请求返回状态码,同REST接口调用返回状态码一致
- public String getMessage()； //请求处理结果说明
- public Object getObject()； //**注意，该对象为返回的内容对象，不同业务类型，代表的实体类型不同，需要使用instanceof等方法进行预判**
- public String getTakeTime()； //业务请求处理时间

#### ICommonInvoker

- Response query(Request request)；//查询接口,参数为实体Request
- String usage(String url)；//使用方法说明调用接口,参数为：资源定位符，例如：/com/nsn/do/mt/hbase/xdr
- String usages() ；//全部使用方法调用接口
- ArrayList<String> getBinsTbls() ；//bin文件字段信息调用接口
- ArrayList<String> getXDRTbls()；//xdr文件字段信息调用接口

## REST

### 调用约定

- 调用方法：GET
- 字符编码：utf-8
- 字符输入：半角
- 回传格式：JSON，使用jackson进行序列化

### 接口

#### 规范接口

- 请求URL：http://mt-server:port/mt
- 规范参数：
  1. imsis,imsis=460113006449246,460113006449246,460113006449246，多个内容请使用半角逗号分隔，如**缺失请传-1**
  2. msisdns,msisdns=8618958834000,8618958834000,8618958834000，多个内容请使用半角逗号分隔，如**缺失请传-1**
  3. startTime,startTime=2019-01-01 23:00:00，请使用该类型的时间格式
  4. endTime,endTime=2019-01-02 23:00:00，请使用该类型的时间格式
  5. tblLimit,tblLimit=s1mme,s1u_http,s1u_dns，多个接口类型，请使用半角逗号分隔
- 请求样例：http://mt-server:port/mt?imsis=460113006449246& msisdns=8618958834000&startTime=2019-01-01 23:00:00&endTime=2019-01-02 23:00:00&tblLimit=s1mme,s1u_http,s1u_dns

#### MTSQL查询接口

- 请求URL：<http://mt-server:port/mt-ext>

- 参       数：

  1. mtSql，mtSql=sql字符串，规避特殊字符的请求限制，需要对请求sql进行hex编码，具体编码样例请参考如下：

     ```
     package com.nsn.mt.http;
     
     import org.apache.commons.codec.DecoderException;
     import org.apache.commons.codec.EncoderException;
     import org.apache.commons.codec.binary.Hex;
     import org.apache.http.HttpEntity;
     import org.apache.http.NameValuePair;
     import org.apache.http.client.ClientProtocolException;
     import org.apache.http.client.entity.UrlEncodedFormEntity;
     import org.apache.http.client.methods.HttpPost;
     import org.apache.http.impl.client.CloseableHttpClient;
     import org.apache.http.impl.client.HttpClients;
     import org.apache.http.message.BasicNameValuePair;
     import org.apache.http.util.EntityUtils;
     import java.io.IOException;
     import java.net.URISyntaxException;
     import java.nio.charset.StandardCharsets;
     import java.util.ArrayList;
     import java.util.List;
     /**
      * author: BISCUIT_0
      * date: 2019/2/28 16:18
      * 请求样例需要：httpclient-4.5.7.jar和commons-codec-1.12.jar三方库支持
      * 设定当前mt server访问地址为：http://localhost:18080
      */
     public class MTExtHttpClientDemo {
         public static void main(String[] args) throws IOException, URISyntaxException, EncoderException, DecoderException {
             String sql = "get * from cmcc_bigxdr_reg,cmcc_bigxdr_call case { (imsi='460008626943542' and local_city!='376') or ( msisdn='8618837508021' or imsi='460008626943542' )},{ msisdn='8618837508021' or imsi='460008626943542'}";
             CloseableHttpClient httpClient = HttpClients.createDefault();
             HttpPost post = new HttpPost("http://localhost:18080/mt-ext");
             List<NameValuePair> params = new ArrayList<NameValuePair>();
             String mtSqlHex= encodeSql(sql);
             params.add(new BasicNameValuePair("mtSql",mtSqlHex));
             params.add(new BasicNameValuePair("startTime", "2019-01-28 00:00:00"));
             params.add(new BasicNameValuePair("endTime", "2019-01-28 23:59:59"));
             System.out.println(mtSqlHex);
             post.setEntity(new UrlEncodedFormEntity(params));
             System.out.println(post.getURI());
             String responseString = httpClient.execute(post, (response) -> {
                 int status = response.getStatusLine().getStatusCode();
                 if (status >= 200 && status < 300) {
                     HttpEntity entity = response.getEntity();
                     return entity != null ? EntityUtils.toString(entity) : null;
                 } else {
                     throw new ClientProtocolException("Unexpected response status: " + status);
                 }
             });
             System.out.println(responseString);
             httpClient.close();
         }
     
         private static String encodeSql(String sql) {
             return sql == null ? "" : Hex.encodeHexString(sql.getBytes(StandardCharsets.UTF_8));
         }
     
         private static String decodeSql(String hexSql) {
             try {
                 return hexSql == null ? "" : new String(Hex.decodeHex(hexSql), StandardCharsets.UTF_8);
             } catch (DecoderException e) {
                 e.printStackTrace();
             }
             return "";
         }
     }
     
     ```

  2. startTime,startTime=2019-01-01 23:00:00，请使用该类型的时间格式

  3. endTime,endTime=2019-01-02 23:00:00，请使用该类型的时间格式


### JSON样例

```
{
    "_ts_":"1545016679638",
    "_tt_":"156223",    
    "resp_code":"000",
    "resp_msg":"Successfully",
    "req_params":{
        "key1":"value1",
        "key2":"value2",
        "key3":"value3",
        "key4":"value4"
    },
    "resp_data":[
            "s1mme":{
                "version":"32",
                "header":[
                    {
                        "name":"iface",
                        "type":"int",
                        "comment":"业务类型"
                    },
                    {
                        "name":"xdr_id",
                        "type":"string",
                        "comment":"XDR_ID"
                    },
                    {
                        "name":"imsi",
                        "type":"string",
                        "comment":"imsi"
                    },
                    {
                        "name":"imei",
                        "type":"string",
                        "comment":"imei"
                    },
                    {
                        "name":"rat",
                        "type":"int",
                        "comment":"rat"
                    },
                    {
                        "name":"home_province",
                        "type":"string",
                        "comment":"省份代码"
                    },
                    ......，
                    {
                        "name":"mt_row_id",
                        "type":"string",
                        "comment":"MT Server内部使用，用于定位数据源，解析时请丢弃"
                    }
                ],
                "rows:[
                    "4|7a8PSlNfa1Itdxwu9za7Wg==|3569790693402703|3824745013|460|0|6301|1|24388",
                    "4|7a8PSlNfa1Itdxwu9za7Wg==|3569790693402703|3824745013|460|0|6301|1|24388",
                    "4|7a8PSlNfa1Itdxwu9za7Wg==|3569790693402703|3824745013|460|0|6301|1|24388",
                    "4|7a8PSlNfa1Itdxwu9za7Wg==|3569790693402703|3824745013|460|0|6301|1|24388",
                ],
                "row_count":4
            },
            "s1u_http":{
                "version":"32",
                "header":[
                    {
                        "name":"iface",
                        "type":"int",
                        "comment":"业务类型"
                    },
                    {
                        "name":"xdr_id",
                        "type":"int",
                        "comment":"XDR_ID"
                    },
                    {
                        "name":"imsi",
                        "type":"int",
                        "comment":"imsi"
                    },
                    {
                        "name":"imei",
                        "type":"int",
                        "comment":"imei"
                    },
                    {
                        "name":"rat",
                        "type":"int",
                        "comment":"rat"
                    },
                    {
                        "name":"home_province",
                        "type":"string",
                        "comment":"省份代码"
                    },
                    .......，
                    {
                        "name":"mt_row_id",
                        "type":"string",
                        "comment":"MT Server内部使用，用于定位数据源，解析时请丢弃"
                    }
                ],
                "rows":[
                    "4|7a8PSlNfa1Itdxwu9za7Wg==|3569790693402703|3824745013|460|0|6301|1|24388",
                    "4|7a8PSlNfa1Itdxwu9za7Wg==|3569790693402703|3824745013|460|0|6301|1|24388",
                    "4|7a8PSlNfa1Itdxwu9za7Wg==|3569790693402703|3824745013|460|0|6301|1|24388",
                    "4|7a8PSlNfa1Itdxwu9za7Wg==|3569790693402703|3824745013|460|0|6301|1|24388",
                ],
                "row_count":4
            },
            ......
        ]
}    

```

### JSON字段说明

- \_ts\_,响应时间戳,每次请求时间戳不同

- \_tt\_,请求处理耗时,单位是毫秒

- resp_code,请求响应代码

- resp_msg,响应代码说明

- req_param,请求参数处理结果

- resp_data,响应数据,数据格式为对象数组

  - 对象（接口）标准名称

    - version,接口数据对应的版本信息，标准版本采用十进制方式，例如，规范为3.2版本设定为：32

    - header,返回数据字段元数据信息，有序列表类型

      - name，字段元数据名称

      - type，类型

      - comment，注解解释说明

        > 针对规范中数组类型的字段，转换规则如下：

        ```
        #例如s1mme数据字段中存在如下的数组信息：
            #代表数据字段内容的长度
        	eps_bearer_number
        	#代表数据字段内容的字段信息
                bearer_id 			                short(2),
                bearer_type 		                short(2),
                bearer_oci 			                short(2),
        		......
        	#数据解析时，按照数据的最长字段设定，约定header中字段下标从0开始
                bearer_id_0
                bearer_type_0
                bearer_oci_0
                ......
                bearer_id_1
                bearer_type_1
                bearer_oci_1
                ......	
        			
        ```

    - rows,返回数据类型，采用竖线分隔方式

    - row_count,返回的记录数

## 数据规范

### 数据类型

 - datetime，Timestamp方式toString转换
 - string，字符串类型
 - int，整形类型
 - short，按照整形类型处理
 - long，长整形数据类型
 - float，浮点数类型
 - double，双精度类型
 - ipv4，转换10进制数字字符串
 - ipv6，转换10进制数字字符串
 - bit，整形类型对应int类型
 - tinyint，整形类型对应int类型
 - smallint，整形类型对应int类型
 - bigint，长整形类型对应long类型
 - real，对应float类型
 - varchar，对应string类型
 - nvarchar，对应string类型
 - binary，对应string类型
 - varbinary，对应string类型

### 空值转换

- 竖线分隔（oracle数据库示意）
  - ||，is null
  - |null|，=‘null’，字符串类型null