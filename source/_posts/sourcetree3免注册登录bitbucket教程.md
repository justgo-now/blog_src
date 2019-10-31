---
title: sourcetree3免注册登录bitbucket教程
date: 2019-10-31 17:10:00
updated: 2019-10-31 17:10:00
category: 
- 开发工具
tags:
- git
- sourcetree
---
##  Sourcetree下载

Sourcetree：一个用于Windows和Mac的免费Git客户端。
官网：[Sourcetree | Free Git GUI for Mac and Windows](https://www.sourcetreeapp.com/)

<!-- more -->

下载之后，双击打开安装程序，新版本安装界面如下，提示需要登录bitbucket账号后才能继续安装


旧版 sourcetree 只需要手动添加 一个 accounts.json 文件就能实现免注册登录，而新版本则需要再修改 user.config 文件中的配置。





## 添加 accounts.json 文件

`%LocalAppData%\Atlassian\SourceTree\accounts.json`
在以上目录位置创建一个accounts.json文件，内容如下：

```json
[
  {
    "$id": "1",
    "$type": "SourceTree.Api.Host.Identity.Model.IdentityAccount, SourceTree.Api.Host.Identity",
    "Authenticate": true,
    "HostInstance": {
      "$id": "2",
      "$type": "SourceTree.Host.Atlassianaccount.AtlassianAccountInstance, SourceTree.Host.AtlassianAccount",
      "Host": {
        "$id": "3",
        "$type": "SourceTree.Host.Atlassianaccount.AtlassianAccountHost, SourceTree.Host.AtlassianAccount",
        "Id": "atlassian account"
      },
      "BaseUrl": "https://id.atlassian.com/"
    },
    "Credentials": {
      "$id": "4",
      "$type": "SourceTree.Model.BasicAuthCredentials, SourceTree.Api.Account",
      "Username": "",
      "Email": null
    },
    "IsDefault": false
  }
]
```



## 修改 user.config 文件配置

该文件所在路径：
`%LocalAppData%\Atlassian\SourceTree.exe_Url_xxxxxxxxxx\3.1.2.3027\user.config`



记事本打开 user.config，在`<SourceTree.Properties.Settings>`添加以下内容并保存即可。

```
<setting name="AgreedToEULAVersion" serializeAs="String">
    <value>20160201</value>
</setting>
```

最后再次打开 sourcetree 安装程序。



