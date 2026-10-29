---
title: "标题：如何搭建入侵检测系统NIDS（Suricata+Elasticsearch+Kibana+Filebeat）"
date: 2026-10-29
description: ""
hiddenInHomeList: false
categories:
  - IT
tags:
  - IT
---

# 标题：如何搭建入侵检测系统NIDS（Suricata+Elasticsearch+Kibana+Filebeat）

标题：如何使用Suricata搭建网络入侵检测系统NIDS_2（Suricata + Elasticsearch + Kibana + Filebeat ）

作者：JeremyLiang/5i双手控球

创建：2024年10月21日

更新：

---

目录：

	⚄ 前言

	⚄ 工具简介及作用

		1）Suricata简介及作用

	⚄ 服务器环境

	⚄ 搭建测试环境

		2）安装Suricata 7.0.7

	⚄ 调试

	⚄ 参考资源

---

⚄ 前言

我写的上一篇文章《如何搭建网络入侵检测系统NIDS（Zeek + Elasticsearch + Kibana + Filebeat ）》用了几天，发现Zeek在记录流量协议方面较优秀，能记录各种协议，缺点是该工具支持的已有告警规则比较少，由于某些特殊原因，此处不展开讲，需要更换NIDS，刚开始的时候考虑过Suricata这款工具，这款工具支持NIDS和NIPS，如果看过我上一篇文章应该能知道刚开始使用了Amazon Linux 2，环境安装出现了各种问题，此处不细讲。经过查阅资料，发现Suricata这款免费且开源的工具支持自定义告警规则，各界用户写好的开源规则也多，官方也在支持更新告警规则，于是选择把Zeek更换为Suricata。

---

⚄ 工具简介及作用

1）Suricata简介及作用

简介:Suricata是一款高性能的开源入侵检测工具，支持自定义编写告警规则，业界开源的告警规则较多。
作用:

监控网络流量，检测潜在的攻击行为，如端口扫描、拒绝服务攻击 (DoS)、缓冲区溢出、Web攻击等。
生成警报和日志，帮助安全人员及时响应安全事件。

---

⚄ 服务器环境

$ cat /etc/issue

Ubuntu 24.04.1 LTS \n \l

⚄ 搭建测试环境

2）安装Suricata 7.0.7参[21]

$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:oisf/suricata-stable
$ sudo apt-get update
$ sudo apt-get install suricata

查看Suricata是否安装成功

$ suricata -V

This is Suricata version 7.0.7 RELEASE

查看本机网卡

$ ifconfig

在Suricata配置文件中设置监听的网卡

$ vim /etc/suricata/suricata.yaml

设置网卡的字段在af-packet下面，找到af-packet在第几行

$ grep -n af-packet /etc/suricata/suricata.yaml

621:af-packet:

af-packet下边是 - interface: ，在该interface写上你自己本机监听流量的网卡

踩坑点：我刚开始跟着，参[20]，更新Suricata免费规则集

$ sudo suricata-update

这个命令下载免费的5w多条规则，下载完之后会默认测试规则是否可用，我的服务器等了十几分钟还没跑完，且机器特别卡，后面只能杀进程。Suricata集成了多种规则集，以下命令会列出各种规则集，这些规则集具体含义可参[22]，

$ suricata-update list-sources

21/10/2024 -- 05:39:58 - <Info> -- Using data-directory /var/lib/suricata.
21/10/2024 -- 05:39:58 - <Info> -- Using Suricata configuration /etc/suricata/suricata.yaml
21/10/2024 -- 05:39:58 - <Info> -- Using /usr/share/suricata/rules for Suricata provided rules.
21/10/2024 -- 05:39:58 - <Info> -- Found Suricata version 7.0.7 at /usr/bin/suricata.
21/10/2024 -- 05:39:58 - <Warning> -- Source index does not exist, will use bundled one.
21/10/2024 -- 05:39:58 - <Warning> -- Please run suricata-update update-sources.
Name: et/open
Vendor: Proofpoint
Summary: Emerging Threats Open Ruleset
License: MIT
Name: et/pro
Vendor: Proofpoint
Summary: Emerging Threats Pro Ruleset
License: Commercial
Replaces: et/open
Parameters: secret-code
Subscription: https://www.proofpoint.com/us/threat-insight/et-pro-ruleset
Name: etnetera/aggressive
Vendor: Etnetera a.s.
Summary: Etnetera aggressive IP blacklist
License: MIT

如果想添加某个规则集，可使用以下命令添加规则集，比如添加ptrules/open

$ suricata-update enable-source ptrules/open

$ suricata-update

使用这个命令下载完3000多条规则后会加上suricata-update添加的5w条规则，Suricata运行测试5w多条规则，大家考虑自己机器情况，否则机器可能会崩溃。

我服务器运行测试5w多条规则跑不动，此方法行不通之后我删除了这5w多条规则，下载了[参22]中的开源ptresearch/attackdetection和默认suricata-update list-sources命令下的Name: ptrules/open做测试。

查看 /etc/suricata/suricata.yaml文件默认加载规则路径在哪（我版本的默认目录是/var/lib/suricata/rules），把告警规则的文件放在/var/lib/suricata/rules下并在/etc/suricata/suricata.yaml的/var/lib/suricata/rules下面rule-files添加告警规则的文件名。我改了规则名的情况如下

```/etc/suricata/suricata.yaml
default-rule-path: /var/lib/suricata/rules

rule-files:
          - test.rules
         - test2.rules


```

测试规则是否可用

$  suricata -T -c /etc/suricata/suricata.yaml -v

Notice: suricata: Configuration provided was successfully loaded. Exiting.

如果加载两个类似的配置文件，修订时间不同，但是规则编号同，这个时候会告警，告警规则有更加新的版本。

如规则文件里面的rev代表着规则的版本，版本数字越大，代表着版本更新

rev4>rev3

启动Suricata

$ systemctl restart suricata

参[20],运行curl http://testmynids.org/uid/index.html生成一个 HTTP 请求，该请求将返回一个匹配 Suricata 警报规则的响应：

上面的规则告警在https://rules.emergingthreats.net/open/suricata-7.0.3/的默认规则库里，我是只摘抄这一条告警规则来做测试。

$ curl http://testmynids.org/uid/index.html

uid=0(root) gid=0(root) groups=0(root)

查看告警日志,如果有以下告警说明搭建成功。

$ cat /var/log/suricata/fast.log

10/21/2024-08:42:45.446204  [1:2100498:7] GPL ATTACK_RESPONSE id check returned root  [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 13.35.185.106:80 -> 10.20.131.239:56196

我下载了参[23]的所有规则再把参[22],建议停用的规则停用，大概是3w多条规则。

---

⚄ 调试

联动elk，elk有Suricata模块，登录web控制台（如果不知道是什么控制台请看我上一篇文章：如何搭建网络入侵检测系统NIDS（Zeek + Elasticsearch + Kibana + Filebeat ）elk的首页点击Add integrations 找到Suricata模块，找到Suricata Events，点击右下角的Also available in Beats、Suricata Logs,Filebaet的安装方式在上一篇文章有（如何搭建网络入侵检测系统NIDS（Zeek + Elasticsearch + Kibana + Filebeat）

filebeat有联动Suricata的模块,配置模块，切换到配置文件目录

$ cd /etc/filebeat/modules.d

修改filebeat的Suricata配置文件，增加suricata的日志文件路径

$ vim suricata.yml.disabled

```代码参考
# Module: suricata
# Docs: https://www.elastic.co/guide/en/beats/filebeat/7.17/filebeat-module-suricata.html

- module: suricata
  # All logs
  eve:
    enabled: true
    var.paths: ["/var/log/suricata/eve.json"]
    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    #var.paths:
```

启用suribata模块

$ filebeat modules enable suricata

在web控制台的elk的首页点击Add integrations 找到Suricata模块，找到Suricata Events，点击右下角的Also available in Beats、Suricata Logs、Check data，如果提示Data successfully received from this module证明配置接入成功。

Data successfully received from this module右下角有一个Suricata Events Overview，该功能模块提供了事件可告警的可视化，点击Alerts即可看到告警事件，使用afrog漏洞扫描工具扫描，Alerts可发现具体的告警。

此篇完。

---

⚄ 参考资源

[20] 安全防护之路——Suricata部署与测试 - Byc4i [2023年-01月-31日]

https://www.freebuf.com/articles/es/353548.html

[21] https://docs.suricata.io/en/suricata-7.0.7/install.html#ubuntu-from-personal-package-archives-ppa

[22] 入侵检测之流量分析--suricata规则库来源梳理 - qianlan [2023年12月12日]

https://mp.weixin.qq.com/s?__biz=Mzg2NTczNDg1MQ==&mid=2247484250&idx=1&sn=098565a030c259b2aeae85ca03db17aa&chksm=ce54daf6f92353e07fcf124f7b4beee8b368780f6c3d66452746a6e193e40e951b243d10103e&scene=178&cur_album_id=3226971432659468288#rd

[23] https://rules.emergingthreats.net/open/suricata-7.0.3/rules/

[24] Suricata规则管理2 - SJG [2021年-06月-18日]

https://juejin.cn/post/6975152890851622919

[25] https://rules.ptsecurity.com/files/ptopen-all#L4

[26] https://rules.evebox.org/rulesets

[27] http://47.108.150.136:8080/IDS/
