---
title: office wopi 
tags: [在线office]
id: ''
categories:

    - php

date: 2020-10-15 14:19:34
---

### PHP实现 MS-WOPI host 简单实现office文档查看/编辑

最近调研一款在线office预览、编辑的方案，看了毕升office、卓正office、冰蓝office这些收费的方案，整体上可以满足需求，不过各有优缺点。最后找到微软官方出品的office online server（原来叫 office web apps），最重要的一点是它完全免费 :rofl:。

#### office online server 部署

office online server 的部署还算比较繁琐，主要原因也是它支持windows服务器，平常大家用的最多的还是Linux，所以多windows的熟悉程度就没有这么得心应手了。

##### 准备工作

1.  必须准备2台服务器，因为OfficeOnline不会在包含 Active Directory 域服务 (AD DS) 的服务器上运行。
2.  域控服务可以使用Windows Server 2016 核心版(无GUI版)，但是Office Online Server 2017 仅支持 Windows Server 2016 的“含桌面体验的服务器”安装选项(如果自行安装的话)。
3.  Office Server 2017的服务器尽可能干净，最好是给一台全新的服务器。
4.  为保证安全性，生产环境中请固定暴露需要使用到的端口。
5.  **请勿在运行 Office Online Server** 的服务器上安装任何其他服务器应用程序。包括 Exchange Server、SharePoint Server、Skype for Business Server 和 SQL Server。如果服务器不足，则可以在这些服务器的其中一台的虚拟机上运行 Office Online Server。
6.  **不要在端口 80、443 或 809 上安装依赖 Web 服务器 (IIS) 角色的任何服务或角色**，因为 Office Online Server 会定期删除这些端口上的 Web 应用程序。
7.  **不要安装任何版本的 Office**。如果已经安装，在安装 Office Online Server 之前必须将其卸载。
8.  **Office Online不会加载IP地址的Doc源, 读取的地址必须是带域名的**. 例如[http://1.2.3.4/t.docx是会报Error的](http://1.2.3.4/t.docx%E6%98%AF%E4%BC%9A%E6%8A%A5Error%E7%9A%84)

   
   具体的安装步骤这边就不做赘述了，网上的案例很多。例如：[https://blog.csdn.net/q386815991/article/details/81705128](https://blog.csdn.net/q386815991/article/details/81705128)

   **注意：** 

   安装往成后在浏览器中输入地址 `http://servername/hosting/discovery` ，显示如下则说明安装成功

[![BkjDOS.md.png](https://s1.ax1x.com/2020/10/23/BkjDOS.md.png)](https://imgchr.com/i/BkjDOS)

``` shell
不过有可能会报错显示error，可能是在部署场的时候没有设置-OpenFromUrlEnabled:$true，在powershell中执行：
Set-OfficeWebAppsFarm -OpenFromUrlEnabled:$true

部署场的命令可以改为：New-OfficeWebAppsFarm -InternalURL "http://servername" -AllowHttp -EditingEnabled -OpenFromUrlEnabled:$true
```

##### 测试结果

然后输入 `http://servername/op/generate.aspx`
[![Bkz3LT.md.png](https://s1.ax1x.com/2020/10/23/Bkz3LT.md.png)](https://imgchr.com/i/Bkz3LT)

输入一个文档链接，生成create link，用浏览器打开即可预览你的文档了。

[![BkzT0S.md.png](https://s1.ax1x.com/2020/10/23/BkzT0S.md.png)](https://imgchr.com/i/BkzT0S)

#### wopi host 实现

> Web Application Open Platform Interface (WOPI). WOPI为基于Web的服务提供了一种查看和编辑OfficeWebApps中文档的方法，您可以从Office文档中获得所有高保真和丰富的文档。

客户端如何使用WOPI的一个例子是为特定类型的文件提供基于浏览器的查看器。该客户端使用WOPI获取文件的内容，以便将该内容以浏览器中的网页形式呈现给用户。下图显示了如何工作的示例。

![](https://oscimg.oschina.net/oscnet/badbe607395f32d1ca972769269c562d6b0.jpg)

图1 使用WOPI为特定类型的文件提供基于浏览器的查看器

上图中交互过程中一个值得注意的细节是，WOPI服务器提供了“调用WOPI客户端所需的信息”。WOPI客户端提供了一种机制，通过该机制，WOPI服务器可以发现WOPI客户端的能力，以及调用这些功能的方法。(正如[部署 Office Online Server](https://my.oschina.net/tita/blog/2963590)中所讲到的../hosting/discovery)下图描述了这种交互：

![](https://oscimg.oschina.net/oscnet/ac043e78db209eb5e0aa385dcfd0477ebce.jpg)

图2 WOPI 发现

> 简单的说我们要实现文件的编辑需要实现三个接口：

GET api/wopi/files/{name} <br>
GET api/wopi/files/{name}/contents<br>
POST api/wopi/files/{name}/contents

##### 接口实现

* 获取文件信息 CheckFileInfo

``` 
    http://server/<...>/wopi*/files/<id>?access_token=<token> 
    get method
    ```

php code：
    

``` php
        static function CheckFileInfo($fileName, $GUID)
            {
                if(isset($_SERVER['DOCUMENT_ROOT'])) {
                    $FileInfoDto = array(
                        'BaseFileName' => $fileName,
                        'OwnerId' => 'admin',
                        // 'ReadOnly' => true,
                        'SHA256' => base64_encode(hash_file('sha256', $_SERVER['DOCUMENT_ROOT'] . '/' . $fileName, true)),
                        'Size' => filesize($_SERVER['DOCUMENT_ROOT'] . '/' . $fileName),
                        'Version' => 1,
                        'UserCanWrite' => true //注意这个参数，为true时才有编辑当前文档的权限
                    );

                    $jsonString = json_encode($FileInfoDto);
                    header('Content-Type: application/json');
                    echo $jsonString;
                }
                else die();
            }
    ```

* 获取文件流  GetFile

  

``` 
  http://server/<...>/wopi*/files/<id>/contents?access_token=<token>
  get method
  ```

  php code：
  

``` php
  static function GetFile($fileName)
    {
        if (file_exists($_SERVER['DOCUMENT_ROOT'] . '/' . $fileName)) {
            header('Content-Description: File Transfer');
            header('Content-Type: application/octet-stream');
            header('Content-Disposition: attachment; filename=' . basename($_SERVER['DOCUMENT_ROOT'] . '/' . $fileName));
            header('Expires: 0');
            header('Cache-Control: must-revalidate');
            header('Pragma: public');
            header('Content-Length: ' . filesize($_SERVER['DOCUMENT_ROOT'] . '/' . $fileName));
            readfile($_SERVER['DOCUMENT_ROOT'] . '/' . $fileName);
            exit;
        }
    }

  ```

  + 保存文件 PutFile url和GetFile一样，但是请求方法（request method）不同

    

``` 
    http://server/<...>/wopi*/files/<id>/contents?access_token=<token>
    post method

    wopi client 获取浏览器编辑的文档的内容，将二进制流发送给该PutFile服务，因此实现的时候，保存该二进制流就行了
    ```

实现思路大致如上，这样就可以对office文档进行在线编辑和预览了。

[![BA95Ox.md.png](https://s1.ax1x.com/2020/10/23/BA95Ox.md.png)](https://imgchr.com/i/BA95Ox)

最后安全方面，可以做限制白名单，指定office online server 能够访问的域名

``` 
New-OfficeWebAppsHost -domain "test.wopi.com"  

限制只能访问test.wopi.com 这个域名实现的wopi服务
```
