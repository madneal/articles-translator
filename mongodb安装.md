## 概述[¶](#overview "Permalink to this headline")

使用本教程在Windows系统上安装MongoDB社区版。

平台支持

从2.2版本开始，MongoDB不再支持Windows XP。请使用之后版本的Windows系统来使用MongoDB的之后发布版本。

重要

如果你使用Windows Server 2008 R2或者Windows 7的任何版本的系统，那么请安装一个[_热修复补丁_](http://support.microsoft.com/kb/2731284)来解决Windows里面映射文件的内存问题
## 需求[¶](#requirements "Permalink to this headline")

MongoDB社区版需要Windows Server 2008 R2,Windows Vista,或者之后的系统版本。.msi安装器包含所有的软件依赖并且会自动更新之前通过.msi文件安装的老版本的MongoDB。

## 获取MongoDB社区版本[¶](#get-mongodb-community-edition "Permalink to this headline")

注意

安装MongoDB3.2之前的版本，请参考相应版本的文档。比如，参考版本[_3.0_](https://docs.mongodb.com/v3.0/tutorial/install-mongodb-on-windows/).

1

### 决定你需要哪一种MongoDB包[¶](#determine-which-mongodb-build-you-need "Permalink to this headline")

下面的MongoDB包能够在Windows上使用：

**MongoDB for Windows 64-bit**只能运行在Window Server 2008 R2，Windows 7 64位以及之后版本的系统。这个安装包得益于Windows平台最近的优化并且不能够在之前的版本里运行。

**MongoDB for Windows 64-bit Legacy** 运行在Windows Vista以及Windows Server 2008并且不包括最近的性能优化。

为了看你在使用哪种版本的Windows，你可以在 _Command Prompt_ 或者 _Powershell_输入以下的命令行：

```
wmic os get caption
wmic os get osarchitecture

```

2

### 对于Windows系统下载MongoDB[¶](#download-mongodb-for-windows "Permalink to this headline")

在[MongoDB下载页面](http://www.mongodb.org/downloads)下载MongoDB最新发布的产品。确保你下载了Windows系统对应的正确的MongoDB版本。64位的MongoDB不能够在32位的Windows系统上运行。


## 安装MongoDB社区版本[¶](#install-mongodb-community-edition "Permalink to this headline")

### 交互式安装[¶](#interactive-installation "Permalink to this headline")

1

#### 对于Windows系统安装MongoDB[¶](#install-mongodb-for-windows "Permalink to this headline")

在资源管理器中，找到下载的MongoDB .msi文件，一般是在默认的下载文件夹下。双击.msi文件。一系列的操作会引导安装流程。

你可以设置安装路径如果你选择“自定义”安装选项。

注意

这份介绍基于你已经将MongoDB安装到 C:\Program Files\MongoDB\Server\3.2。

MongoDB是自我包含的并且没有其他的系统依赖。你可以在任何文件夹内运行MongoDB。你可以在任何文件中安装MongoDb(e.g. D:\test\mongodb)

### 自动化安装[¶](#unattended-installation "Permalink to this headline")

你可以在命令行里面使用msiexec.exe来安装MongoDB社区版。

1

#### 打开管理员命令行提示符[¶](#open-an-administrator-command-prompt "Permalink to this headline")

按下Win键，输入cmd.exe，接着按下Ctrl + Shift + Enter从而作为管理员运行命令行提示符。

在管理员命令行提示符里面执行其它的步骤。

2

#### 对于Windows安装MongoDB[¶](#id2 "Permalink to this headline")

切换到包含.msi安装二进制文件的文件夹并且调用：
```
msiexec.exe /q /i mongodb-win32-x86_64-2008plus-ssl-3.4.1-signed.msi ^
            INSTALLLOCATION="C:\Program Files\MongoDB\Server\3.4.1\" ^
            ADDLOCAL="all"

```

你可以通过过修改INSTALLOCATION值来制定安装位置。

默认这个方法会安装MongoDB所有的二进制文件。为了安装特定的MongoDB组件集合，你可以制定他们通过添加ADDLOCAL参数，参数的格式是分号隔开的序列，这个序列包含一个或者多个下列的组件集合:

**组件集合**
**二进制文件**

服务器
mongod.exe

路由
mongos.exe

客户端
mongo.exe

监测工具
mongostat.exe, mongotop.exe

导入导出工具
mongodump.exe, mongorestore.exe, mongoexport.exe, mongoimport.exe

其它工具
bsondump.exe, mongofiles.exe, mongooplog.exe, mongoperf.exe

比如，如果只想安装MongoDB工具，则调用：

```
msiexec.exe /q /i mongodb-win32-x86_64-2008plus-ssl-3.4.1-signed.msi ^
            INSTALLLOCATION="C:\Program Files\MongoDB\Server\3.4.1\" ^
            ADDLOCAL="MonitoringTools,ImportExportTools,MiscellaneousTools"

```

## 运行MongoDB社区版[¶](#run-mongodb-community-edition "Permalink to this headline")

警告

不要在没有认证设置里面没有设置“安全模式”的情况下让[mongod.exe](../../reference/program/mongod.exe/#bin.mongod.exe)在外网可见。MongoDB是被设计用于在可信任的环境中运行，数据不会默认开启“安全模式”。

1

### 设置MongoDB环境[¶](#set-up-the-mongodb-environment "Permalink to this headline")

MongoDB需要一个[_data directory_](../../reference/glossary/#term-dbpath)来存储所有的数据。MongoDB的默认数据目录路径是你启动MongoDB磁盘里面的绝对路径\data\db。通过在命令行提示符运行下面的命令来创建这个文件夹：

```
md \data\db

```

你也可以使用--dbpath选项制定数据文件的路径，比如：

```
"C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe" --dbpath d:\test\mongodb\data

```

如果你的路径包含空格，则应该将整个路径放在双引号里面，比如：

```
"C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe" --dbpath "d:\test\mongo db data"

```

你也可以通过[配置文件](../../reference/configuration-options/)来制定dbpath。

2

### 启动MongoDB[¶](#start-mongodb "Permalink to this headline")

为了启动MongoDB,运行 [mongod.exe](../../reference/program/mongod.exe/#bin.mongod.exe)。比如在命令行提示符里：

```
"C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe"

```

这启动了主要的MongoDB数据库进程。稍后出现的连接信息表明[mongod.exe](../../reference/program/mongod.exe/#bin.mongod.exe) 进程已经成功运行。

取决于你系统的安全等级，对于C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe与网络通信时候的“某些特性”，系统会弹出安全警告对话框。所有的用户应该选择专有网络，比如我家或者工作网络，并且点击允许。对于安全以及MongoDB的其他信息，可以参考[安全文档](../../security/)

3

### 连接MongoDB[¶](#connect-to-mongodb "Permalink to this headline")

通过[mongo.exe](../../reference/program/mongo/#bin.mongo "mongo") shell来连接MongoDB,打开另外一个命令行提示符。

```
"C:\Program Files\MongoDB\Server\3.4\bin\mongo.exe

```

如果你想开发使用.NET的应用，参考文档[C# and MongoDB](https://docs.mongodb.com/ecosystem/drivers/csharp)来获取更多信息。

4

### 开始使用MongoDB[¶](#begin-using-mongodb "Permalink to this headline")

为了帮助你开始使用MongoDB,MongoDB在各个版本提供了[入门指导](../../#getting-started)。在 [_Getting Started_](../../#getting-started) 里面可以看到现有的版本。

在将MongoDB部署到产品环境之前，阅读[_Production Notes_](../../administration/production-notes/)文档。

之后，想暂停MongoDB的话，在运行[mongod](../../reference/program/mongod/#bin.mongod "mongod")实例的终端里面按Control+C。

## 为MongoDB社区版配置Windows服务[¶](#configure-a-windows-service-for-mongodb-community-edition "Permalink to this headline")

1

### 打开管理员命令行提示符[¶](#id3 "Permalink to this headline")

按下Win键，输入cmd.exe，接着按下Ctrl + Shift + Enter来运行以管理员身份运行命令行提示符。

在管理员命令行提示符里面运行以下步骤。

2

### 创建目录[¶](#create-directories "Permalink to this headline")

为你的数据库和日志文件创立文件夹：

```
mkdir c:\data\db
mkdir c:\data\log

```

3

### 创建配置文件[¶](#create-a-configuration-file "Permalink to this headline")

创建配置文件。这个文件**必须**设置[systemLog.path](../../reference/configuration-options/#systemLog.path "systemLog.path")，也包含其他的[_configuration options_](../../reference/configuration-options/)。

比如，创建文件C:\Program Files\MongoDB\Server\3.4\mongod.cfg来指定[systemLog.path](../../reference/configuration-options/#systemLog.path "systemLog.path")以及 [storage.dbPath](../../reference/configuration-options/#storage.dbPath "storage.dbPath")：

```
systemLog:
    destination: file
    path: c:\data\log\mongod.log
storage:
    dbPath: c:\data\db

```

4

### 安装MongoDB服务[¶](#install-the-mongodb-service "Permalink to this headline")

重要

以“管理员权限”在命令行提示符里面运行下面所有的命令：

通过以--install选项启动[mongod.exe](../../reference/program/mongod.exe/#bin.mongod.exe)来安装MongoDB服务，并且通过-config选项来指定之前创建的配置文件。

```
"C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe" --config "C:\Program Files\MongoDB\Server\3.4\mongod.cfg" --install

```

为了指定dbpath，你可已在配置文件里面设置路径(e.g. C:\mongodb\mongod.cfg)或者在命令行立案通过--dbpath选项来设置。

如果需要的话，你可以对[mongod.exe](../../reference/program/mongod.exe/#bin.mongod.exe)或者[mongos.exe](../../reference/program/mongos.exe/#bin.mongos.exe)的多个实例安装服务。对于每一个服务，设置一个特别的--serviceName以及--serviceDisplayName。只有在你系统资源允许并且你的系统设计需要这样的情况下才使用多个实例。

5

### 启动MongoDB服务[¶](#start-the-mongodb-service "Permalink to this headline")

```
net start MongoDB

```

6

### 停止或者移除MongoDB服务[¶](#stop-or-remove-the-mongodb-service-as-needed "Permalink to this headline")

使用下面的命令停止MongoDB服务：

```
net stop MongoDB

```

使用下面的命令移除MongoDB服务：

```
"C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe" --remove

```

## 为MongoDB社区版手动创建Windows服务[¶](#manually-create-a-windows-service-for-mongodb-community-edition "Permalink to this headline")

你可以将MongoDB服务器设置为Windows服务令其在启动的时候自动运行。

下面的流程是基于假设你已经通过.msi安装器在路径 C:\Program Files\MongoDB\Server\3.2里安装了MongoDB社区版。

如果你已经在其他的目录安装过了，你可能需要调整路径。

1

### 打开管理员命令行提示符[¶](#id5 "Permalink to this headline")

按下Win键，输入cmd.exe，接着按下Ctrl + Shift + Enter来运行以管理员身份运行命令行提示符。

在管理员命令行提示符里面运行以下步骤。

2

### 创建文件夹[¶](#id7 "Permalink to this headline")

为你的数据库和日志文件创立文件夹：

```
mkdir c:\data\db
mkdir c:\data\log

```

3

### 创建配置文件[¶](#id9 "Permalink to this headline")

创建配置文件。这个文件**必须**设置[systemLog.path](../../reference/configuration-options/#systemLog.path)，也包含其他的[_configuration options_](../../reference/configuration-options/)。

比如，创建文件C:\Program Files\MongoDB\Server\3.4\mongod.cfg来指定[systemLog.path](../../reference/configuration-options/#systemLog.path)以及 [storage.dbPath](../../reference/configuration-options/#storage.dbPath)：

```
systemLog:
    destination: file
    path: c:\data\log\mongod.log
storage:
    dbPath: c:\data\db

```

4

### 安装MongoDB服务[¶](#create-the-mongodb-service "Permalink to this headline")

创建MongoDB服务：
```
sc.exe create MongoDB binPath= "\"C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe\" --service --config=\"C:\Program Files\MongoDB\Server\3.4\mongod.cfg\"" DisplayName= "MongoDB" start= "auto"

```

sc.exe需要在“=”和配置项值之间设置空格(eg “binPath= ”)，并且通过“\”来转义双引号。

如果服务成功创建，会出现下面的日志信息：

```
[SC] CreateService SUCCESS

```

5

### 启动MongoDB服务.[¶](#id11 "Permalink to this headline")

```
net start MongoDB

```

6

### 停止或者移除MongoDB服务[¶](#id13 "Permalink to this headline")

使用下面的命令停止MongoDB服务：

```
net stop MongoDB

```

为了移除MongoDB服务，首先暂停服务接着运行以下命令：

```
sc.exe delete MongoDB

```

## 其它资源[¶](#additional-resources "Permalink to this headline")

*   [MongoDB开发人员免费课程](https://university.mongodb.com/courses/M101P/about?jmp=docs)

*   [MongoDB.NET 开发人员免费在线课程](https://university.mongodb.com/courses/M101N/about?jmp=docs)

*   [MongoDB 架构指导](https://www.mongodb.com/lp/white-paper/architecture-guide?jmp=docs)

[](https://twitter.com/MongoDB)
[](https://www.youtube.com/user/MongoDB)
[](https://www.facebook.com/mongodb)
[](https://plus.google.com/u/1/101024085748034940765/posts?cfem=1)
​                