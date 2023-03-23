# HEKA

## heka介绍

Heka 是一个”瑞士军刀”级别的流式数据处理工具，由 Mozilla 开源，有如下功能：

- 加载解析日志文件。
- 接收 Statsd 类型的指标数据进行合并，上载至时间序列数据库，如 graphite 或 InfluxDB。
- 启动扩展进程来收集本地系统的操作数据。
- 实时分析、画图，并能够对经过 Heka 数据管道的数据进行异常检测。
- 通过像 AMQP 或 TCP等协议将数据从一处传输至另一处。
- 将处理后的结果数据存储至一个或多个持久化数据库。

> heka的数据流向input--->splitter--->decoder--->router--->filter--->output
>
> hekad 是核心进程，单个 hekad 进程可以配置很多插件，同时处理多种数据的收集、处理、传输工作
>
> -version 参数查看版本号。
>
> -config=<config_path>。指定配置文件。默认位置为 /etc/hekad.toml，此配置可以指定目录，hekad 会解析读取目录下的所有配置文件

## heka安装

这里我们采用二进制文件安装方式

从github下下载heka的最新安装包https://github.com/mozilla-services/heka/releases

解压到/opt目录

```shell
tar -C /opt -xzvf heka-0_10_0-linux-amd64.tar.gz
ln -s heka-0_10_0-linux-amd64 heka
```

配置环境变量

```
vi /etc/profile
export HEKA_HOME=/opt/heka
export PATH=$HEKA_HOME/bin:$PATH
```

创建配置文件

```shell
touch /$HEKA_HOME/config/centos.toml

[audit-log-input]
type = "LogstreamerInput"
log_directory = "/var/log/audit"
file_match = 'audit\.log'
#splitter = "audit-log-splitter"

[audit-log-splitter]
type = "RegexSplitter"
delimiter = '\s+'
delimiter_eol = false

[audit-log-encoder]
append_newlines = true
type = "PayloadEncoder"
prefix_ts = false

[audit-log-output]
type = "UdpOutput"
address = "bsa:7777"
message_matcher = "TRUE"
encoder = "audit-log-encsheoder"

```

## Heka的配置

> Heka 的配置文件采用 TOML 格式，使用 [] 来区分一段段配置

## 常用插件

参考https://hekad.readthedocs.io/en/v0.10.0/config/splitters/

### Inputs插件

> Input 插件从外部获取数据，并将其传入 heka 管道；数据的来源可以是本地文件系统、远程服务器、socket等的结构或非结构化数据。此插件只能使用 Go 语言编写

#### Logstreamer

**hostname** (String) - 在同一主机上可能会有多个heka的input在一个配置文件中，为了避免导致混淆，可以设置该名称用于区分不同的input，非必填

**oldest_duration** (String) - 文件的修改事件早于持续事件，则文件将不进行处理，默认为720小时（30天），可以设置如“2s”、“2m”、“2h”。

**journal_directory** (string) - 读取文明数据的跟踪文件用于跟踪到目前为止已读取到的位置,用于存储日志文件的目录，默认情况下，它存储在heka的基本目录下

**log_directory** (string) - 扫描文件的根目录，扫描为递归扫描

**rescan_interval** (int) - 检查文件存在的频率，启动heka时候文件不存在，可能在后续才会创建该文件，heka将根据这个间隔重新扫描，默认值为5秒

**file_match** (string) - 匹配位于log目录下文件的正则表达式，这个表达式末尾会自动田间$,并将log_directory作为前缀，提示：文件匹配通常应该用单引号分隔，表示使用了原始字符串，而不是双引号，双引号要求转义所有反斜杠。例如，‘access\\.log’会正常工作，但"access\\.log"不会，如果使用双引号，则需要采用这样的方式"access\\\\.log"才能实现相同的结果。

**priority** (list of strings) - 当使用顺序日志流时,如log.1，log.2，log.3，优先级是如何按从最旧到最新的顺序对日志文件进行排序

**differentiator** (list of strings) - 当使用多个logstreams时，微分器是一组字符串，将用于记录器的命名，与文件_match中捕获的组匹配的部分将在中替换其匹配值。

**translation** (hash map of hash maps of ints) - 一组翻译映射，用于将匹配的分组转换为整数，以用于排序。

**splitter** (string, optional) - 默认设置为“TokenSplitter”，它会将日志流拆分为每行一条Heka消息。可以自定义splitter插件。

### Splitter插件

> Splitter 将接收到的数据分隔成有效的记录，如使用换行分隔；只能使用 Go 语言编写。

#### Common Splitter

splitter插件是通用的配置选项,在所有的Splitter中都由这些配置属性：

**keep_truncated** (bool, optional) - 如果为true，则任何超过输入缓冲区容量的记录仍将以截断形式传递。如果为false，则这些记录将被删除。默认为false，超出了缓冲区，截断也没有什么意义了，这里不用修改。

**use_message_bytes** (bool, optional):为true时做字节块转换，消息写入MsgBytes，默认为false，作为字符串写入消息的payload，如果要做ProtobufDecoder转换时需要注意这块。

**min_buffer_size** (uint, optional) - 缓冲数据流的内部缓冲区的初始大小，不能大于全局配置的最大消息大小。默认值为8KiB，某些Splitter可能会设置不同的默认值。

**deliver_incomplete_final** (bool, optional): 默认为false，还需要研究使用场景

#### HekaFramingSplitter

用于拆分heka创建的流数据，在编码区的消息头支持HMAC密钥身份认证，认证时候，如果认证失败，则消息会自动丢弃掉，**HekaFramingSplitter**需要和**ProtobufDecoder**一起使用

**signer** - 可选选项，由签名者姓名、下划线和密钥的数字版本组成，其中以**.**分割，如acl_splitter.signer.ops_0，acl_splitter.signer.ops_1

```ini
[acl_splitter.signer.ops_0]
hmac_key = "4865ey9urgkidls xtb0[7lf9rzcivthkm"
[acl_splitter.signer.ops_1]
hmac_key = "xdd908lfcgikauexdi8elogusridaxoalf"

[acl_splitter.signer.dev_1]
hmac_key = "haeoufyaiofeugdsnzaogpi.ua,dp.804u"
```

**use_message_bytes** (bool, optional) - 默认为true，为true时做字节块转换，消息写入MsgBytes，默认为false，作为字符串写入消息的payload,属于splitter的通用配置。

**skip_authentication** (bool, optional) -  默认为false，跳过认证，如在从文件系统加载protobug编码的Heka消息时，文件中不带密钥信息，则需要设置为true，跳过认证环节，一般线上很少做认证

例子：

```ini
[acl_splitter]
type = "HekaFramingSplitter"
  [acl_splitter.signer.ops_0]
  hmac_key = "4865ey9urgkidls xtb0[7lf9rzcivthkm"
  [acl_splitter.signer.ops_1]
  hmac_key = "xdd908lfcgikauexdi8elogusridaxoalf"

  [acl_splitter.signer.dev_1]
  hmac_key = "haeoufyaiofeugdsnzaogpi.ua,dp.804u"

[tcp_control]
type = "TcpInput"
address = ":5566"
splitter = "acl_splitter"
```

#### NullSplitter

在传入的数据已经自然划分，不需要进行再次划分时候使用，如在UDP消息传入时候，每一个UDP数据包的内容将被转换为一个单独的消息。NullSplitter 没有配置选项，它不需要单独的 TOML 配置部分。

#### RegexSplitter

RegexSplitter通过正则匹配组的方式匹配输入信息，如果指定了匹配组，则捕获的文本将包含在返回记录中，如果不是，则范围的记录将不包含正则表达式的匹配文本。

**delimiter** (string) - 正则表达式

**delimiter_eol** (bool, optional) - 是否识别正则表达式 捕获组

```ini
[mysql_slow_query_splitter]
type = "RegexSplitter"
delimiter = '\n(# User@Host:)'
delimiter_eol = false
```



### Decoders插件

> Decoder 插件将接收到的数据解析成结构化数据。可使用 Go 来写或者 lua code。

#### SandboxDecoder

> SandboxDecoder为数据解析和复杂转换提供了一个独立的执行环境，而无需重新编译Heka。，在heka中的沙盒解码器中封装了一些常用的解码规则，这些解码规则采用lua语言实现，脚本在$HEKA_HOME/share/heka/lua_decoders目录下，其中包括以下的lua脚本:

##### Apache Access Log Decoder

> 根据Apache“LogFormat”配置指令解析Apache访问日志。Apache格式说明符映射到适用的Nginx变量名上，例如%a->remote_addr。这允许通用web过滤器和输出与任何HTTP服务器输入一起工作
>
> lua脚本：**lua_decoders/apache_access.lua**

**log_format** (string) - apache2的LogFormat配置指令。参见http://httpd.apache.org/docs/2.4/mod/mod_log_config.html

**type** (string, optional, default nil) - 将消息“Type”字段设置为指定值

**user_agent_transform** (bool, optional, default false)

**user_agent_keep** (bool, optional, default false)

**user_agent_conditional** (bool, optional, default false)

**payload_keep** (bool, optional, default false)

##### graylog_decoder



apache_access.lua     json.lua             linux_loadavg.lua   linux_netdev.lua   linux_procstat.lua      mysql_slow_query.lua  nginx_error.lua        rsyslog.lua
graylog_extended.lua  linux_diskstats.lua  linux_memstats.lua  linux_netstat.lua  mariadb_slow_query.lua  nginx_access.lua      nginx_stub_status.lua

#### GeoIpDecoder

**db_file** - GeoLiteCity.dat数据库文件位置，默认会从/var/cache/hekad/GeoLiteCity.dat处加载

**source_ip_field** - 需要进行转换的IP地址的字段的名称

**target_field** - 解码器创建的新字段的名称。解码器将输出包含以下元素的JSON对象

- latitute: string - 纬度
- longitude - 经度
- location - 位置
- coordinates - 坐标
- countrycode - 国家代码
- countrycode3
- region - 区域(省)
- city - 城市
- postalcode - 邮编码
- areacode -  区域编号
- charset - 字符集
- continentalcode - 还不清楚

例：

```ini
[apache_geoip_decoder]
type = "GeoIpDecoder"
db_file="/etc/geoip/GeoLiteCity.dat"
source_ip_field="remote_host"
target_field="geoip"
```



### Filter插件

> Filter 插件是Heka 的处理引擎。接收匹配规则的的数据；用来监控、聚集统计或处理数据。可以用 Go、或 lua。使用 Lua 开发，可以在不重启 heka 服务的前提下，将插件注入至运行时服务。

### Encoders插件

> 是 decoder 的反向处理工具；内嵌在 Output 插件中，相当于序列化。可以使用 Go 或lua 开发。

### Output插件

> 按照匹配规则将序列化后的数据输出到目标中。仅能使用Go编写。

#### UdpOutput

**net** - 通信的网络类型，必须是udp | udp4 | udp6 | unixgram，默认为udp

**address** - ip地址:端口

**local_address** - 用于生成的数据报数据包的本地地址

**encoder** - 指定encoder插件的名称，encoder插件需要在配置文件中进行配置

**max_message_size**  - 允许通过UdpOutput发送的最大消息大小，默认值为65507
