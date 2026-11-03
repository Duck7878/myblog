---
title: "Windows系统后门-隐藏账户"
date: 2026-11-03
description: ""
hiddenInHomeList: false
categories:
  - 攻防技术
tags:
  - 攻防技术
---

# Windows系统后门-隐藏账户

创建：2022年11月03日

Review: 2026-10-30 21:32:43

# 1. 技术介绍

## 1.1 简介

创建本地账户（T1136.001 - ATT&CK）归属于权限维持（Persistence，TA0003）下的创建账户（T1136）。

![1667457599974-2e721a62-63c3-4cef-b162-c272384a7a3d.png](assets/1667457599974-2e721a62-63c3-4cef-b162-c272384a7a3d-070512-20260506173205-ep64zj1.png)

攻击者可以创建一个本地帐户来维持对受害系统的访问。net user /add命令可用于创建本地帐户。在macOS系统上，该dscl -create命令可用于创建本地帐户。

## 1.2 使用范围

Windows系统。

## 1.3 攻击限制

需获取主机执行权限或者主机Administrator账户凭证。

# 2 攻击视角

## 2.1 攻击场景

1.攻击前序：攻击者拥有了目标主机的执行权限，或者获取了Administrator账户凭证

2.攻击后续：攻击者可以窃取系统凭证信息

- Credential Access —— T1003 OS Credential Dumping

![1667458913587-3a534fc5-4232-4b6e-86b4-e63db2a545b9.png](assets/1667458913587-3a534fc5-4232-4b6e-86b4-e63db2a545b9-739662-20260506173205-153xkbr.png)

## 2.2 攻击复现

复现环境信息

|主机类型|windows主机|
| ------------| -------------------------|
|操作系统版本|Windows 10 Server 2012 r2|
|IP地址|192.168.1.1|
|账号名称|Administrator|
|账号权限|Administrator|

攻击步骤

```plain
net user yincang$ Test1234! /add #添加yincang$隐藏用户
net localgroup administrators yincang$ /add #将yincang$用户添加进管理员组中
```

![1667461532148-a228c2f0-0c02-4ff6-83a6-a5c32ca10f4a.png](assets/1667461532148-a228c2f0-0c02-4ff6-83a6-a5c32ca10f4a-718418-20260506173205-yhs3bje.png)

注意一：  
此时虽然使用命令行无法看到 yincang 隐藏用户，但是采用其它方法仍旧可以发现yincang隐藏用户，为了更好的隐藏新建的账户，还需要进行修改注册表文件操作。

![1667461627864-72774f92-ab55-4429-9a8b-fa63274a1c1d.png](assets/1667461627864-72774f92-ab55-4429-9a8b-fa63274a1c1d-889862-20260506173205-bthsz3n.png)

注意二：

1、打开windws server 2012的远程桌面功能

2、需将新创建的隐藏账户 yincang 添加在允许远程桌面用户的位置，默认只允许 Administrator，不然隐藏账户yincang无法登录。  

3、远程登录隐藏账户 yincang账户名是yincang密码是Test1234!

![1667461809630-331873fc-7ba9-46aa-bba0-2a29bd87d532.png](assets/1667461809630-331873fc-7ba9-46aa-bba0-2a29bd87d532-626681-20260506173205-i47uv3p.png)

‍

查看yincang$ 隐藏用户方法一：

通过 控制面板 用户账户 管理账户 依然还是可以看到 yincang$ 账户存在的。

![1667466387980-55cbf447-b268-4f53-bf87-05d5d708b8fb.png](assets/1667466387980-55cbf447-b268-4f53-bf87-05d5d708b8fb-181470-20260506173205-ccdfdys.png)

查看yincang$ 隐藏用户方法二：

或行输入命令：compmgmt.msc，即可打开计算机管理查看用户。

![1667466405751-5211858e-447f-4b18-bf9e-7045eea5d901.png](assets/1667466405751-5211858e-447f-4b18-bf9e-7045eea5d901-149862-20260506173205-xrm22o8.png)

cmd输入命令即可打开注册表:  **regedt32 **

首先打开注册表编辑器，找到HKEY_LOCAL_MACHINE\SAM\SAM，点击右键，选择“权限”，将Administrator用户的权限，设置成“完全控制”，然后重新打开注册表，确保可以看到SAM路径下的文件。

![1667463396567-6a9a3ddf-448e-4c28-be5b-0bc7b90e79e6.png](assets/1667463396567-6a9a3ddf-448e-4c28-be5b-0bc7b90e79e6-195257-20260506173205-d930o88.png)

重新打开注册表，确保可以看到SAM路径下的文件

其次前往SAM/Domains/Account/Users/Names处，选择Administrator用户，在右侧的键值处可以找到对应的值如1f4，然后从左侧的Users目录下可以找到对应的文件。

![1667463530750-00006d87-c037-4f72-b137-0dd79157a9cb.png](assets/1667463530750-00006d87-c037-4f72-b137-0dd79157a9cb-569309-20260506173205-iihsn1n.png)

然后从对应的000001F4文件中将键值对F的值复制出来。然后同理找到隐藏账户yincang$所对应的文件000003EC，并将从Administrator文件中复制出来的F值粘贴进 yincang$所对应的000003EC的F值中。

![1667466436362-ee46f5a4-0086-4a7c-9b8c-321792ef6ac1.png](assets/1667466436362-ee46f5a4-0086-4a7c-9b8c-321792ef6ac1-945094-20260506173205-84u59xi.png)

![1667463644621-d04db243-78db-4729-8ccc-3cd3ee49693c.png](assets/1667463644621-d04db243-78db-4729-8ccc-3cd3ee49693c-393188-20260506173205-yyun7pl.png)

最后将yincang $和000003EC从注册表中右键导出，并删除yincang$用户，然后将刚刚导出的两个文件重新导入进注册表中即可实现yincang用户的隐藏。

删除用户命令

```plain
net user yincang$ /del
```

![1667463912363-a6fff6a9-850b-4cd0-aeef-c292e5f36d96.png](assets/1667463912363-a6fff6a9-850b-4cd0-aeef-c292e5f36d96-792172-20260506173205-3vdwm8r.png)

点击刚刚导出保存的 yincang和000003EC，点击之后会自动导入注册表，下图是导入成功的

![1667463992814-a7db25d6-ab4e-45f5-8570-70c2bd8f61b9.png](assets/1667463992814-a7db25d6-ab4e-45f5-8570-70c2bd8f61b9-576802-20260506173205-16tapha.png)

此时 yincang$隐藏账户在控制面板，用户账户，管理账户，管理工具，计算机管理 ，本地用户和组用户均未发现，成功隐藏

![1667465740031-a1523c84-1504-416d-ab96-0701be06368a.png](assets/1667465740031-a1523c84-1504-416d-ab96-0701be06368a-214715-20260506173205-igz1r99.png)

隐藏成功之后把SAM的权限取消。

![1667466506891-4742504f-e6c3-4330-a876-7694306ebd87.png](assets/1667466506891-4742504f-e6c3-4330-a876-7694306ebd87-205024-20260506173205-mk0tso8.png)

# 3 应急响应发现

输入命令：regedt32 打开注册表

首先打开注册表编辑器，找到HKEY_LOCAL_MACHINE\SAM\SAM，点击右键，选择“权限”，将Administrator用户的权限，设置成“完全控制”，然后重新打开注册表，确保可以看到SAM路径下的文件。

查看注册表，在注册表中 HKEY_LOCAL_MACHINE\SA\SAM\Domains\Account\Users\Names 位置可以看到所有的用户名，将这些用户名与控制面板，用户账户，管理账户，管理工具，计算机管理，本地用户和组用户进行对比，不存在的账户就是隐藏账户。

![1667465708022-411744ea-0cdc-4bb8-9702-605e8c070792.png](assets/1667465708022-411744ea-0cdc-4bb8-9702-605e8c070792-588627-20260506173205-5ysdzix.png)

‍
