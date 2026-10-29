---
title: "标题：如何搭建网络入侵检测系统NIDS（Zeek + Elasticsearch + Kibana + Filebeat ）"
date: 2026-10-29
description: ""
hiddenInHomeList: false
categories:
  - IT
tags:
  - IT
---

# 标题：如何搭建网络入侵检测系统NIDS（Zeek + Elasticsearch + Kibana + Filebeat ）

标题：如何搭建网络入侵检测系统NIDS（Zeek + Elasticsearch + Kibana + Filebeat ）

作者：JeremyLiang/5i双手控球

创建：2024年10月14日

更新：

---

目录：

	⚄ 前言

	⚄ 工具简介及作用

		1）Zeek简介及作用

		2）Elasticsearch简介及作用

		3）Kibana简介及作用

		4）Filebeat简介及作用

	⚄ 服务器环境

	⚄ 搭建测试环境

		5）安装Zeek 7.0.3

		6）安装Elasticsearch 7.17.24

		7）安装Kibana 7.17.24

		8）安装Filebeat 7.17.24

	⚄ 调试

	⚄ 参考资源

---

⚄ 前言

网络入侵检测系统（Network Intrusion Detection System，简称NIDS）是一种关键的网络安全工具，它能够实时监控网络流量，检测并报告任何可疑活动或潜在的安全威胁。通过部署NIDS，安全运营人员可以及时发现并响应各种网络攻击，从而增强其整体安全防护能力。在本次教程中，我将使用Zeek捕获和分析网络流量，生成详细的日志记录，再用Filebeat 从 Zeek 生成的日志文件中读取数据，并将其发送到 Elasticsearch，再使用Elasticsearch 存储和索引来自 Filebeat 的日志数据，最后使用Kibana 将 Elasticsearch 中的数据可视化展示，提供直观的监控和分析界面。用户可以通过 Kibana 创建仪表板和警报，实时监控网络活动和威胁。我将详细的写出涵盖从安装到配置的全过程，参[19],因为网上的资料每一项都有如过江之鲫的现成资料可获取，但那是别人的知识，学完并写完，就变成自己的知识，好记性不如烂笔头，认真记录这一次搭建NIDS系统的过程并分享给大家，把经验、教训、猜测、疑问、踩坑点，统统记下来。

---

⚄ 工具简介及作用

1）Zeek简介及作用

简介:Zeek（以前称为Bro）是一个强大的开源网络入侵检测系统（NIDS），它不仅能够检测网络攻击，还能进行详细的网络流量分析。
作用:
网络监控: Zeek 可以捕获和分析网络流量，生成详细的日志记录。
威胁检测: 通过内置的脚本和规则，Zeek 能够检测潜在的安全威胁和异常行为。
数据收集: 收集各种类型的网络数据，包括HTTP请求、DNS查询、SMTP邮件等。

2）Elasticsearch简介及作用

简介:Elasticsearch 是一个分布式搜索和分析引擎，基于 Apache Lucene 构建，广泛用于全文搜索、日志分析、实时应用监控等领域。
作用:
全文搜索: 提供高效的全文搜索功能，支持复杂的查询和过滤。用户可以通过简单的API调用进行复杂的搜索操作。
数据存储: 存储大量结构化和非结构化的数据。Elasticsearch 的分布式特性使其能够处理海量数据。
实时分析: 提供实时的数据分析能力，支持聚合、统计等功能。用户可以对数据进行实时的统计和分析。
高可用性和可扩展性: 支持分布式部署，具备高可用性和水平扩展能力。这使得 Elasticsearch 适用于大规模的企业级应用。

3）Kibana简介及作用

简介:Kibana 是一个开源的数据可视化工具，专门用于与 Elasticsearch 配合使用，提供直观的数据展示和交互界面。
作用:
数据可视化: 通过图表、仪表板等形式，将 Elasticsearch 中的数据可视化展示。用户可以创建各种图表和仪表板来展示数据。
实时监控: 实时监控系统状态，展示关键指标和趋势。Kibana 提供了丰富的监控功能，帮助用户实时了解系统状态。
日志分析: 提供日志分析功能，帮助用户快速查找和分析日志数据。用户可以通过 Kibana 查看和搜索日志文件。
报告生成: 生成和导出报告，方便分享和存档。Kibana 支持生成各种格式的报告，便于用户分享和存档。

4）Filebeat简介及作用

简介:Filebeat 是 Elastic Stack 的一部分，用于收集和转发日志文件数据到 Elasticsearch 或 Logstash。
作用:
日志收集: 从文件系统中读取日志文件，并将其发送到指定的目标。Filebeat 支持多种日志文件格式。
轻量级: 设计为轻量级且资源占用低，适合在生产环境中部署。Filebeat 的资源消耗非常小，不会对系统性能造成影响。
模块化: 支持多种预配置的模块，如 Zeek、Nginx、Syslog 等。用户可以轻松配置 Filebeat 来收集特定类型的日志。
可靠传输: 提供可靠的日志传输机制，确保数据不会丢失。Filebeat 使用持久化队列来保证数据的可靠性。

---

⚄ 服务器环境

$ cat /etc/issue

Ubuntu 24.04.1 LTS \n \l

---

⚄ 搭建测试环境

5）安装Zeek 7.0.3

踩坑点：刚开始我使用的是Amazon Linux 2安装zeek，参[1]，安装方式有3种，Amazon Linux 2没有二进制包，选择了源码编译安装方式，该方式需要安装各种依赖（某些依赖由于Amazon Linux 2的yum包安装不了，只能通过下载源代码编译，编译过程中又缺少依赖，死循环），最后安装完Zeek的依赖之后编译花了大约120分钟，编译时间较久，最后Zeek成功运行，后来换另一台一台机器重新安装Zeek，各种报错、各种需要安装依赖，报错太多，此处不细讲，最后的解决方式是去更换一台能添加软件源并手动安装的方式，参[2],参[3],的方式进行安装。

$ echo 'deb http://download.opensuse.org/repositories/security:/zeek/xUbuntu_24.04/ /' | sudo tee /etc/apt/sources.list.d/security:zeek.list
$ curl -fsSL https://download.opensuse.org/repositories/security:zeek/xUbuntu_24.04/Release.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/security_zeek.gpg > /dev/null
$ sudo apt update
$ sudo apt install zeek-7.0

执行过程中如果弹窗询问你是否配置邮箱通知，我选择No configuration

查看/opt/zeek路径下是否有文件，此处注意（如果使用编译方式安装Zeek，那Zeek路径在/usr/local/zeek，如果使用添加源或者下载二进制包的方式安装Zeek，那Zeek路径在/opt/zeek）

$ ls /opt/zeek

将/opt/zeek/bin目录加到PATH环境变量中

$echo &quot;PATH=$PATH:/opt/zeek/bin" >> /etc/profile

加载配置文件

$ source /etc/profile

输出环境变量查看/opt/zeek/bin是否在环境变量中

$echo$PATH

输出Zeek版本

$ zeek -v

查看本机网卡

$ ifconfig

将网卡设置成混杂模式

$ ifconfig ensXXXXX promisc

zeek配置文件配置修改监听网卡

$ vim /opt/zeek/etc

修改interface为当前实际网卡

[zeek]

type=standalone

host=localhost

Interface=ensXXXXX

此处来自官方介绍：在某些情况下，最好以json格式呈现日志，因为json格式更容易解析，如果需要将日志发送到ek日志平台，日志格式必须修改为json格式。

zeek的配置文件修改日志格式为json格式

$ vim /opt/zeek/share/zeek/site/local.zeek

在任意位置处新增以下代码

@load policy/tuning/json-logs.zeek

参[4],使用ZeekControl管理Zeek

启动zeek

$ zeekctl

$ deploy

查看日志目录，可看到已经生成日志

$ cd /opt/zeek/logs/current

参[4],notice.log，用来辨认出Zeek认为可能是有趣、奇怪或者恶意的特定活动。在Zeek的语言中，这样的活动被称作 “notice”。

参[18],https://github.com/jqlang/jq这个工具助于查看生成的json格式日志。

使用方法

$  jq . http.log

筛选固定参数语句

$ jq -c '[."id.orig_h", ."query", ."answers"]' dns.log

6）安装Elasticsearch 7.17.24

截止至2024年10月11日,elasticsearch和kibana软件7.x最新版本为7.17

$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.17.24-linux-x86_64.tar.gz

$ tar zxvf elasticsearch-7.17.24-linux-x86_64.tar.gz

elasticsearch 必须使用非root用户运行,创建用户

$ adduser elasticsearch

将elasticsearch文件的所有权更改为 'elasticsearch' 用户

$ chown -R elasticsearch elasticsearch-7.17.24

移动到用户的主目录并设置权限

$ cp -r elasticsearch-7.17.24 /home/elasticsearch/

$ cd /home/elasticsearch/

$ chown -R elasticsearch elasticsearch-7.17.24/

启动elasticsearch

$ /home/elasticsearch/elasticsearch-7.17.24/bin/elasticsearch

检测是否启动成功

$ curl localhost:9200

出现elasticsearch版本号为7.17.24证明运行成功。

以上是未授权即可访问，为了增加安全性添加账号密码。

$ vim /home/elasticsearch/elasticsearch-7.17.24/config/elasticsearch.yml

任意空白处增加以下代码

xpack.security.enabled: true

配置文件修改配置之后访问系统，系统显示访问失败，说明配置密码成功

$ curl localhost:9200

设置初始化密码，注意设置初始化密码必须在elasticsearch运行状态下才可运行。

$ /home/elasticsearch/elasticsearch-7.17.24/bin/elasticsearch-setup-passwords auto

记录生成的账号密码

用户名elastic是http://ip:5601的系统账号密码

用户名kibana是和elasticsearch通讯的账号密码（填写在kibana的配置文件里）

使用生成的账号密码进行测试是否可用

$ curl localhost:9200 -u elastic:密码

7）安装Kibana 7.17.24

$ wget https://artifacts.elastic.co/downloads/kibana/kibana-7.17.24-linux-x86_64.tar.gz

$ tar zxvf kibana-7.17.24-linux-x86_64.tar.gz

$ chown -R elasticsearch kibana-7.17.24-linux-x86_64

$ cp -r kibana-7.17.24-linux-x86_64 /home/elasticsearch/

$ cd /home/elasticsearch/

$ chown -R elasticsearch kibana-7.17.24-linux-x86_64/

切换到elasticsearch用户

$ su elasticsearch

修改kibana配置文件

$ vim /home/elasticsearch/kibana-7.17.24-linux-x86_64/config/kibana.yml

配置elasticsearch的账号密码，取消server.port的注释，修改server.host主机：0.0.0.0，127.0.0.1或者localhost外部无法访问，取消elasticsearch.hosts的注释，增加账号密码

先运行elassticsearch 再运行kibana，访问该服务器对应的5601端口。

踩坑点：前一天还能正常运行，后一天启动kibana时报错，报错时的配置文件

---

```
# Kibana is served by a back end server. This setting specifies the port to use.
server.port: 5601

# Specifies the address to which the Kibana server will bind. IP addresses and host names are both valid values.
# The default is 'localhost', which usually means remote machines will not be able to connect.
# To allow connections from remote users, set this parameter to a non-loopback address.
server.host: "0.0.0.0"

# Enables you to specify a path to mount Kibana at if you are running behind a proxy.
# Use the `server.rewriteBasePath` setting to tell Kibana if it should remove the basePath
# from requests it receives, and to prevent a deprecation warning at startup.
# This setting cannot end in a slash.
#server.basePath: ""

# Specifies whether Kibana should rewrite requests that are prefixed with
# `server.basePath` or require that they are rewritten by your reverse proxy.
# This setting was effectively always `false` before Kibana 6.3 and will
# default to `true` starting in Kibana 7.0.
#server.rewriteBasePath: false

# Specifies the public URL at which Kibana is available for end users. If
# `server.basePath` is configured this URL should end with the same basePath.
#server.publicBaseUrl: ""

# The maximum payload size in bytes for incoming server requests.
#server.maxPayload: 1048576

# The Kibana server's name.  This is used for display purposes.
#server.name: "your-hostname"

# The URLs of the Elasticsearch instances to use for all your queries.
elasticsearch.hosts: ["http://localhost:9200"]

# Kibana uses an index in Elasticsearch to store saved searches, visualizations and
# dashboards. Kibana creates a new index if the index doesn't already exist.
#kibana.index: ".kibana"

# The default application to load.
#kibana.defaultAppId: "home"

# If your Elasticsearch is protected with basic authentication, these settings provide
# the username and password that the Kibana server uses to perform maintenance on the Kibana
# index at startup. Your Kibana users still need to authenticate with Elasticsearch, which
# is proxied through the Kibana server.
elasticsearch.username: "kibana"
elasticsearch.password: ""

# Kibana can also authenticate to Elasticsearch via "service account tokens".
# If may use this token instead of a username/password.
# elasticsearch.serviceAccountToken: "my_token"

# Enables SSL and paths to the PEM-format SSL certificate and SSL key files, respectively.
# These settings enable SSL for outgoing requests from the Kibana server to the browser.
#server.ssl.enabled: false
#server.ssl.certificate: /path/to/your/server.crt
#server.ssl.key: /path/to/your/server.key

# Optional settings that provide the paths to the PEM-format SSL certificate and key files.
# These files are used to verify the identity of Kibana to Elasticsearch and are required when
# xpack.security.http.ssl.client_authentication in Elasticsearch is set to required.
#elasticsearch.ssl.certificate: /path/to/your/client.crt
#elasticsearch.ssl.key: /path/to/your/client.key

# Optional setting that enables you to specify a path to the PEM file for the certificate
# authority for your Elasticsearch instance.
#elasticsearch.ssl.certificateAuthorities: [ "/path/to/your/CA.pem" ]

# To disregard the validity of SSL certificates, change this setting's value to 'none'.
elasticsearch.ssl.verificationMode: none

# Time in milliseconds to wait for Elasticsearch to respond to pings. Defaults to the value of
# the elasticsearch.requestTimeout setting.
#elasticsearch.pingTimeout: 1500

# Time in milliseconds to wait for responses from the back end or Elasticsearch. This value
# must be a positive integer.
#elasticsearch.requestTimeout: 30000

# List of Kibana client-side headers to send to Elasticsearch. To send *no* client-side
# headers, set this value to [] (an empty list).
#elasticsearch.requestHeadersWhitelist: [ authorization ]

# Header names and values that are sent to Elasticsearch. Any custom headers cannot be overwritten
# by client-side headers, regardless of the elasticsearch.requestHeadersWhitelist configuration.
#elasticsearch.customHeaders: {}

# Time in milliseconds for Elasticsearch to wait for responses from shards. Set to 0 to disable.
#elasticsearch.shardTimeout: 30000

# Logs queries sent to Elasticsearch. Requires logging.verbose set to true.
#elasticsearch.logQueries: false

# Specifies the path where Kibana creates the process ID file.
#pid.file: /run/kibana/kibana.pid

# Enables you to specify a file where Kibana stores log output.
#logging.dest: stdout

# Set the value of this setting to true to suppress all logging output.
#logging.silent: false

# Set the value of this setting to true to suppress all logging output other than error messages.
#logging.quiet: false

# Set the value of this setting to true to log all events, including system usage information
# and all requests.
#logging.verbose: false

# Set the interval in milliseconds to sample system and process performance
# metrics. Minimum is 100ms. Defaults to 5000.
#ops.interval: 5000

# Specifies locale to be used for all localizable strings, dates and number formats.
# Supported languages are the following: English - en , by default , Chinese - zh-CN .
#i18n.locale: "en"
```

---

运行正常kibana.yml配置文件如下，这两个配置文件的区别是

取消elasticsearch.ssl.verificationMode的注释并且修改为以下

elasticsearch.ssl.verificationMode: none，参[17],修改之后运行正常

---

```kibana.yml配置文件
# Kibana is served by a back end server. This setting specifies the port to use.
server.port: 5601

# Specifies the address to which the Kibana server will bind. IP addresses and host names are both valid values.
# The default is 'localhost', which usually means remote machines will not be able to connect.
# To allow connections from remote users, set this parameter to a non-loopback address.
server.host: "0.0.0.0"

# Enables you to specify a path to mount Kibana at if you are running behind a proxy.
# Use the `server.rewriteBasePath` setting to tell Kibana if it should remove the basePath
# from requests it receives, and to prevent a deprecation warning at startup.
# This setting cannot end in a slash.
#server.basePath: ""

# Specifies whether Kibana should rewrite requests that are prefixed with
# `server.basePath` or require that they are rewritten by your reverse proxy.
# This setting was effectively always `false` before Kibana 6.3 and will
# default to `true` starting in Kibana 7.0.
#server.rewriteBasePath: false

# Specifies the public URL at which Kibana is available for end users. If
# `server.basePath` is configured this URL should end with the same basePath.
#server.publicBaseUrl: ""

# The maximum payload size in bytes for incoming server requests.
#server.maxPayload: 1048576

# The Kibana server's name.  This is used for display purposes.
#server.name: "your-hostname"

# The URLs of the Elasticsearch instances to use for all your queries.
elasticsearch.hosts: ["http://localhost:9200"]

# Kibana uses an index in Elasticsearch to store saved searches, visualizations and
# dashboards. Kibana creates a new index if the index doesn't already exist.
#kibana.index: ".kibana"

# The default application to load.
#kibana.defaultAppId: "home"

# If your Elasticsearch is protected with basic authentication, these settings provide
# the username and password that the Kibana server uses to perform maintenance on the Kibana
# index at startup. Your Kibana users still need to authenticate with Elasticsearch, which
# is proxied through the Kibana server.
elasticsearch.username: "kibana"
elasticsearch.password: ""

# Kibana can also authenticate to Elasticsearch via "service account tokens".
# If may use this token instead of a username/password.
# elasticsearch.serviceAccountToken: "my_token"

# Enables SSL and paths to the PEM-format SSL certificate and SSL key files, respectively.
# These settings enable SSL for outgoing requests from the Kibana server to the browser.
#server.ssl.enabled: false
#server.ssl.certificate: /path/to/your/server.crt
#server.ssl.key: /path/to/your/server.key

# Optional settings that provide the paths to the PEM-format SSL certificate and key files.
# These files are used to verify the identity of Kibana to Elasticsearch and are required when
# xpack.security.http.ssl.client_authentication in Elasticsearch is set to required.
#elasticsearch.ssl.certificate: /path/to/your/client.crt
#elasticsearch.ssl.key: /path/to/your/client.key

# Optional setting that enables you to specify a path to the PEM file for the certificate
# authority for your Elasticsearch instance.
#elasticsearch.ssl.certificateAuthorities: [ "/path/to/your/CA.pem" ]

# To disregard the validity of SSL certificates, change this setting's value to 'none'.
elasticsearch.ssl.verificationMode: none

# Time in milliseconds to wait for Elasticsearch to respond to pings. Defaults to the value of
# the elasticsearch.requestTimeout setting.
#elasticsearch.pingTimeout: 1500

# Time in milliseconds to wait for responses from the back end or Elasticsearch. This value
# must be a positive integer.
#elasticsearch.requestTimeout: 30000

# List of Kibana client-side headers to send to Elasticsearch. To send *no* client-side
# headers, set this value to [] (an empty list).
#elasticsearch.requestHeadersWhitelist: [ authorization ]

# Header names and values that are sent to Elasticsearch. Any custom headers cannot be overwritten
# by client-side headers, regardless of the elasticsearch.requestHeadersWhitelist configuration.
#elasticsearch.customHeaders: {}

# Time in milliseconds for Elasticsearch to wait for responses from shards. Set to 0 to disable.
#elasticsearch.shardTimeout: 30000

# Logs queries sent to Elasticsearch. Requires logging.verbose set to true.
#elasticsearch.logQueries: false

# Specifies the path where Kibana creates the process ID file.
#pid.file: /run/kibana/kibana.pid

# Enables you to specify a file where Kibana stores log output.
#logging.dest: stdout

# Set the value of this setting to true to suppress all logging output.
#logging.silent: false

# Set the value of this setting to true to suppress all logging output other than error messages.
#logging.quiet: false

# Set the value of this setting to true to log all events, including system usage information
# and all requests.
#logging.verbose: false

# Set the interval in milliseconds to sample system and process performance
# metrics. Minimum is 100ms. Defaults to 5000.
#ops.interval: 5000

# Specifies locale to be used for all localizable strings, dates and number formats.
# Supported languages are the following: English - en , by default , Chinese - zh-CN .
#i18n.locale: "en"
```

---

http://x.x.x.x:5601登录系统

8）安装Filebeat 7.17.24

登录web系统之后，系统主页有一个添加集成,全局搜索Zeek，进入该功能模块，再进入右下角的Zeek Logs功能模块

安装filebaet，选择Linux的DEB

$ curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.17.24-amd64.deb

$ sudo dpkg -i filebeat-7.17.24-amd64.deb

修改filebeat.yml的配置文件

$ vim /etc/filebeat/filebeat.yml

增加zeek日志路径的代码(/opt/zeek/logs/current/*.log),增加Elasticsearch Output的账号密码和取消hosts: [localhost:9200]注释，成功运行配置文件如下

---

```filebeat.yml配置文件
###################### Filebeat Configuration Example #########################

# This file is an example configuration file highlighting only the most common
# options. The filebeat.reference.yml file from the same directory contains all the
# supported options with more comments. You can use it as a reference.
#
# You can find the full configuration reference here:
# https://www.elastic.co/guide/en/beats/filebeat/index.html

# For more available modules and options, please see the filebeat.reference.yml sample
# configuration file.

# ============================== Filebeat inputs ===============================

filebeat.inputs:

# Each - is an input. Most options can be set at the input level, so
# you can use different inputs for various configurations.
# Below are the input specific configurations.

# filestream is an input for collecting log messages from files.
- type: filestream

  # Unique ID among all inputs, an ID is required.
  id: my-filestream-id

  # Change to true to enable this input configuration.
  enabled: false

  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    #- /var/log/*.log
    - /opt/zeek/logs/current/*.log
    #- c:\programdata\elasticsearch\logs\*

  # Exclude lines. A list of regular expressions to match. It drops the lines that are
  # matching any regular expression from the list.
  #exclude_lines: ['^DBG']

  # Include lines. A list of regular expressions to match. It exports the lines that are
  # matching any regular expression from the list.
  #include_lines: ['^ERR', '^WARN']

  # Exclude files. A list of regular expressions to match. Filebeat drops the files that
  # are matching any regular expression from the list. By default, no files are dropped.
  #prospector.scanner.exclude_files: ['.gz$']

  # Optional additional fields. These fields can be freely picked
  # to add additional information to the crawled log files for filtering
  #fields:
  #  level: debug
  #  review: 1

# ============================== Filebeat modules ==============================

filebeat.config.modules:
  # Glob pattern for configuration loading
  path: ${path.config}/modules.d/*.yml

  # Set to true to enable config reloading
  reload.enabled: false

  # Period on which files under path should be checked for changes
  #reload.period: 10s

# ======================= Elasticsearch template setting =======================

setup.template.settings:
  index.number_of_shards: 1
  #index.codec: best_compression
  #_source.enabled: false


# ================================== General ===================================

# The name of the shipper that publishes the network data. It can be used to group
# all the transactions sent by a single shipper in the web interface.
#name:

# The tags of the shipper are included in their own field with each
# transaction published.
#tags: ["service-X", "web-tier"]

# Optional fields that you can specify to add additional information to the
# output.
#fields:
#  env: staging

# ================================= Dashboards =================================
# These settings control loading the sample dashboards to the Kibana index. Loading
# the dashboards is disabled by default and can be enabled either by setting the
# options here or by using the `setup` command.
#setup.dashboards.enabled: false

# The URL from where to download the dashboards archive. By default this URL
# has a value which is computed based on the Beat name and version. For released
# versions, this URL points to the dashboard archive on the artifacts.elastic.co
# website.
#setup.dashboards.url:

# =================================== Kibana ===================================

# Starting with Beats version 6.0.0, the dashboards are loaded via the Kibana API.
# This requires a Kibana endpoint configuration.
setup.kibana:

  # Kibana Host
  # Scheme and port can be left out and will be set to the default (http and 5601)
  # In case you specify and additional path, the scheme is required: http://localhost:5601/path
  # IPv6 addresses should always be defined as: https://[2001:db8::1]:5601
   host: "0.0.0.0:5601"

  # Kibana Space ID
  # ID of the Kibana Space into which the dashboards should be loaded. By default,
  # the Default Space will be used.
  #space.id:

# =============================== Elastic Cloud ================================

# These settings simplify using Filebeat with the Elastic Cloud (https://cloud.elastic.co/).

# The cloud.id setting overwrites the `output.elasticsearch.hosts` and
# `setup.kibana.host` options.
# You can find the `cloud.id` in the Elastic Cloud web UI.
#cloud.id:

# The cloud.auth setting overwrites the `output.elasticsearch.username` and
# `output.elasticsearch.password` settings. The format is `<user>:<pass>`.
#cloud.auth:

# ================================== Outputs ===================================

# Configure what output to use when sending the data collected by the beat.

# ---------------------------- Elasticsearch Output ----------------------------
output.elasticsearch:
  # Array of hosts to connect to.
  hosts: ["localhost:9200"]

  # Protocol - either `http` (default) or `https`.
  #protocol: "https"

  # Authentication credentials - either API key or username/password.
  #api_key: "id:api_key"
  username: "elastic"
  password: ""

# ------------------------------ Logstash Output -------------------------------
#output.logstash:
  # The Logstash hosts
  #hosts: ["localhost:5044"]

  # Optional SSL. By default is off.
  # List of root certificates for HTTPS server verifications
  #ssl.certificate_authorities: ["/etc/pki/root/ca.pem"]

  # Certificate for SSL client authentication
  #ssl.certificate: "/etc/pki/client/cert.pem"

  # Client Certificate Key
  #ssl.key: "/etc/pki/client/cert.key"

# ================================= Processors =================================
processors:
  - add_host_metadata:
      when.not.contains.tags: forwarded
  - add_cloud_metadata: ~
  - add_docker_metadata: ~
  - add_kubernetes_metadata: ~

# ================================== Logging ===================================

# Sets log level. The default log level is info.
# Available log levels are: error, warning, info, debug
#logging.level: debug

# At debug level, you can selectively enable logging only for some components.
# To enable all selectors use ["*"]. Examples of other selectors are "beat",
# "publisher", "service".
#logging.selectors: ["*"]

# ============================= X-Pack Monitoring ==============================
# Filebeat can export internal metrics to a central Elasticsearch monitoring
# cluster.  This requires xpack monitoring to be enabled in Elasticsearch.  The
# reporting is disabled by default.

# Set to true to enable the monitoring reporter.
#monitoring.enabled: false

# Sets the UUID of the Elasticsearch cluster under which monitoring data for this
# Filebeat instance will appear in the Stack Monitoring UI. If output.elasticsearch
# is enabled, the UUID is derived from the Elasticsearch cluster referenced by output.elasticsearch.
#monitoring.cluster_uuid:

# Uncomment to send the metrics to Elasticsearch. Most settings from the
# Elasticsearch output are accepted here as well.
# Note that the settings should point to your Elasticsearch *monitoring* cluster.
# Any setting that is not set is automatically inherited from the Elasticsearch
# output configuration, so if you have the Elasticsearch output configured such
# that it is pointing to your Elasticsearch monitoring cluster, you can simply
# uncomment the following line.
#monitoring.elasticsearch:

# ============================== Instrumentation ===============================

# Instrumentation support for the filebeat.
#instrumentation:
    # Set to true to enable instrumentation of filebeat.
    #enabled: false

    # Environment in which filebeat is running on (eg: staging, production, etc.)
    #environment: ""

    # APM Server hosts to report instrumentation results to.
    #hosts:
    #  - http://localhost:8200

    # API Key for the APM Server(s).
    # If api_key is set then secret_token will be ignored.
    #api_key:

    # Secret token for the APM Server(s).
    #secret_token:


# ================================= Migration ==================================

# This allows to enable 6.7 migration aliases
#migration.6_to_7.enabled: true
```

---

编辑filebeat的module.d模块下的zeek.yml插件

以下能运行成功并记录日志的配置文件

---

```zeek.yml
# Module: zeek
# Docs: https://www.elastic.co/guide/en/beats/filebeat/7.17/filebeat-module-zeek.html

- module: zeek
  capture_loss:
    enabled: false
  connection:
    enabled: false
  dce_rpc:
    enabled: false
  dhcp:
    enabled: false
  dnp3:
    enabled: false
  dns:
    enabled: true
    var.paths: ["/opt/zeek/logs/current/dns.log"]
  dpd:
    enabled: false
  files:
    enabled: false
  ftp:
    enabled: false
  http:
    enabled: true
    var.paths: ["/opt/zeek/logs/current/http.log"]
  intel:
    enabled: false
  irc:
    enabled: false
  kerberos:
    enabled: false
  modbus:
    enabled: false
  mysql:
    enabled: false
  notice:
    enabled: true
    var.paths: ["/opt/zeek/logs/current/notice.log"]
  ntp:
    enabled: false
  ntlm:
    enabled: false
  ocsp:
    enabled: false
  pe:
    enabled: false
  radius:
    enabled: false
  rdp:
    enabled: false
  rfb:
    enabled: false
  signature:
    enabled: false
  sip:
    enabled: false
  smb_cmd:
    enabled: false
  smb_files:
    enabled: false
  smb_mapping:
    enabled: false
  smtp:
    enabled: false
  snmp:
    enabled: false
  socks:
    enabled: false
  ssh:
    enabled: true
    var.paths: ["/opt/zeek/logs/current/ssh.log"]
  ssl:
    enabled: false
  stats:
    enabled: false
  syslog:
    enabled: false
  traceroute:
    enabled: false
  tunnel:
    enabled: false
  weird:
    enabled: false
  x509:
    enabled: false

    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    #var.paths:
```

---

踩坑点：如果zeek.yml没设置指定路径，那么filebeat会去检索以前Zeek前身名字Bro的路径

启动zeek.yml模块

$ sudo filebeat modules enable zeek

检查模块是否启动

$ filebeat modules list

初始化配置文件

$ sudo filebeat setup

启动filebeat,如果启动有日志证明启动成功

$ filebeat -e

⚄ 调试

依次启动zeek、elasticsearch、kibana、filebeat

踩坑点：filebeat的在systemctl是看不了状态的，只能ps | aux grep filebeat查看

web系统Zeek Logs 模块点击check data,查看是否检测到zeek模块

长期使用必须后台保持运行

$ nohup 命令 &

使用流量攻击工具，比如使用fscanssh暴力破解22端口

在web界面的discover功能模块筛选zeek.yml配置模块的规则，即可看到完整攻击流量日志。

event.module: "zeek"

经过实际测试，使用998个密码爆破，web界面显示998条日志，证明可完整记录。

流量机器启动一个http监听，使用dirsearch目录扫描11730个路径进行扫描。

经过实际测试，web界面显示12018条,说明基本都已经记录。

以上NIDS的搭建已经完成并运行，在此基础上可以做的事情就更多了，本篇文章在此做抛砖引玉的作用。

⚄ 参考资源

[1] https://zeek-docs-cn.readthedocs.io/zh-cn/chinese/install/install.html

[2] https://github.com/zeek/zeek/wiki/Binary-Packages

[3] https://software.opensuse.org//download.html?project=security%3Azeek&package=zeek-7.0

[4] https://zeek-docs-cn.readthedocs.io/zh-cn/chinese/quickstart/

[5] 【实践】开源IDS网络流量分析与监控系统Zeek对接GrayLog - yuanfan2012 [2023-02-23]

	https://cloud.tencent.com/developer/article/2222834

[6] 流量分析的瑞士军刀：Zeek - madneal [2020-05-08]

	https://www.freebuf.com/sectool/235587.html

[7] zeek集群部署介绍 - safest_place [2023年08月03日]

	https://mp.weixin.qq.com/s?__biz=MzkzMDE3ODc1Mw==&mid=2247486668&idx=1&sn=b2b08554b097e06b33e7266c8402e0df&chksm=c27f7f62f508f67406288e69a03e2edf5258ac2eb00c67250500707e14483fe11ac3507da624&scene=178&cur_album_id=2143956083483181060#rd

[8] Zeek使用与实践探索二 - safest_place [2023年11月23日]

https://mp.weixin.qq.com/s?__biz=MzkzMDE3ODc1Mw==&mid=2247487058&idx=1&sn=b9edd35388e25c090cfb4aabb4e5c9b1&chksm=c27f7dfcf508f4ea3c1e9477bed089c894dab1717ad7d2540446a76740c0d83c46a2a6697d83&scene=178&cur_album_id=2143956083483181060#rd

[9] Zeek使用与实践探索三 - 最安全的地方 [2024年04月23日]

https://mp.weixin.qq.com/s?__biz=MzkzMDE3ODc1Mw==&mid=2247487629&idx=1&sn=feed6ba9ec0cad4216fe9673b29a6405&chksm=c27f6323f508ea358512fee313a40dc8882eee9297562f05b10840d492a967eb14e82d6a3633&scene=178&cur_album_id=2143956083483181060#rd

[10] zeek系列之：流量数据采集流量探针zeek安装部署 - 超超超超子 [2021年08月11日]

https://blog.csdn.net/g5703129/article/details/119596714

[11] 将 Zeek 与 ELK 栈集成 - Tridev Reddy 译者：LCTT geekpi [2022年06月28日]

https://linux.cn/article-14770-1.html

[12] ELK的安装及简单使用 - Jason's [2022年08月23日]

http://www.884358.com/elk-install/#an_zhuangLogstash833

[13] Sending Zeek logs to ELK using Filebeats - Cyber Tool Guardian [2023年09月23日]

https://medium.com/@cybertoolguardian/sending-zeek-logs-to-elk-using-filebeats-c66b4bea35a4

[14] 在CentOS上安装Elasticsearch和Kibana - demo1234567 [2024年05月-14日]

[15] 借助 VPC Traffic Mirroring 构建网络入侵检测系统 - AWS Team [2020年07月10日]

https://aws.amazon.com/cn/blogs/china/using-vpc-traffic-mirroring-to-construct-network-intrusion-detection-system-update/

[16] 基于 Gateway Load Balancer 的一种 VPC 边界流量镜像方案与实现 -  AWS Team [2023年10月18日]

https://aws.amazon.com/cn/blogs/china/a-vpc-boundary-traffic-mirroring-solution-and-implementation-based-on-gateway-load-balancer/

[17] Unable to retrieve version information from Elasticsearch nodes. connect ETIMEDOUT - Stephen Brown [2024年03月27日]

https://discuss.elastic.co/t/error-elasticsearch-service-unable-to-retrieve-version-information-from-elasticsearch-nodes-connect-etimedout/356120/11

[18] https://zeek-docs-cn.readthedocs.io/zh-cn/chinese/

[19] HTTPS抓包扯淡 - scz [2023年09月01日]

https://scz.617.cn/network/202309010819.txt
