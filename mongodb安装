## 概述[¶](#overview "Permalink to this headline")

Use this tutorial to install MongoDB Community Edition on Windows systems.
使用这个教程在Windows系统上安装MongoDB社区版本。

Platform Support
平台支持

Starting in version 2.2, MongoDB does not support Windows XP. Please
use a more recent version of Windows to use more recent releases of
MongoDB.
从2.2版本开始，MongoDB不会再支持Windows XP。请使用更新版本的Windows系统来使用MongoDB的最新版本。

Important
重要

If you are running any edition of Windows Server 2008
R2 or Windows 7, please install [a hotfix to resolve an issue with
memory mapped files on Windows](http://support.microsoft.com/kb/2731284).
如果你使用Windows Server 2008 r2或者Windows 7的任何版本的系统，那么请安装一个[热修复补丁](http://support.microsoft.com/kb/2731284)来解决Windows里面映射文件的内存问题
## 需求[¶](#requirements "Permalink to this headline")

MongoDB Community Edition requires Windows Server 2008 R2, Windows Vista, or
later. The .msi installer includes all other software dependencies
and will automatically upgrade any older version of MongoDB installed
using an .msi file.
MongoDB社区版需要Windows Server 2008 R2,Windows Vista,或者之后的系统版本。.msi安装器包含所有的软件依赖并且会自动更新之前通过.msi文件安装的老的MongoDB版本。

## 获取MongoDB社区版本[¶](#get-mongodb-community-edition "Permalink to this headline")

Note
注意

To install a version of MongoDB prior to 3.2, please refer to that version’s
documentation. For example, see version 
安装MongoDB3.2之前的版本，请参考相应版本的文档。比如，参考版本[3.0](https://docs.mongodb.com/v3.0/tutorial/install-mongodb-on-windows/).

1

### 决定你需要哪一种MongoDB包[¶](#determine-which-mongodb-build-you-need "Permalink to this headline")

The following MongoDB builds are available for Windows:
下面的MongoDB包能够在Windows上使用：


**MongoDB for Windows 64-bit** runs only
on Windows Server 2008 R2, Windows 7 64-bit, and newer versions of
Windows. This build takes advantage of recent enhancements to the
Windows Platform and cannot operate on older versions of Windows.
**MongoDB for Windows 64-bit**只能运行在Window Server 2008 R2，Windows 7 64位以及更新版本的系统。这个安装包得益于Windows平台最近的优化并且不能够在之前的版本里运行。

**MongoDB for Windows 64-bit Legacy** runs on Windows Vista, and
Windows Server 2008 and does not include recent performance
enhancements.
**MongoDB for Windows 64-bit Legacy** 运行在Windows Vista以及Windows Server 2008并且不报最近的性能优化。

To find which version of Windows you are running, enter the following
commands in the _Command Prompt_ or _Powershell_:
为了看你在使用哪种版本的Windows，你可以在 _Command Prompt_ 或者 _Powershell_输入以下的命令行：

```
wmic os get caption
wmic os get osarchitecture

```

2

### 对于Windows系统下载MongoDB[¶](#download-mongodb-for-windows "Permalink to this headline")

Download the latest production release of MongoDB from the [MongoDB
downloads page](http://www.mongodb.org/downloads). Ensure you download
the correct version of MongoDB for your Windows system. The 64-bit
versions of MongoDB do not work with 32-bit Windows.
在[MongoDB下载页面](http://www.mongodb.org/downloads)下载MongoDB最新发布的产品。确保你下载了Windows系统对应的正确的MongoDB版本。64位的MongoDB不能够在32位的Windows系统上运行。


## 安装MongoDB社区版本[¶](#install-mongodb-community-edition "Permalink to this headline")

### 交互式安装[¶](#interactive-installation "Permalink to this headline")

1

#### 对于Windows系统安装MongoDB[¶](#install-mongodb-for-windows "Permalink to this headline")

In Windows Explorer, locate the downloaded MongoDB .msi file, which
typically is located in the default Downloads folder. Double-click
the .msi file. A set of screens will appear to guide you through the
installation process.
在资源管理器中，找到下载的MongoDB .msi文件，一般是在默认的下载文件夹下。双击.msi文件。一系列的操作会引导安装流程。

You may specify an installation directory if you choose the “Custom”
installation option.
你可以设置安装路径如果你选择“自定义”安装选项。

Note
注意

These instructions assume that you have installed MongoDB
to C:\Program Files\MongoDB\Server\3.2.
这份介绍基于你已经将Mongo。DB安装到 C:\Program Files\MongoDB\Server\3.2。

MongoDB is self-contained and does not have any other system
dependencies. You can run MongoDB from any folder you choose. You may
install MongoDB in any folder (e.g. D:\test\mongodb).
MongoDB是自我包含的并且没有其他的系统依赖。你可以在任何文件夹内运行MongoDB。你可以在任何文件中安装MongoDb(e.g. D:\test\mongodb)

### 自动化安装[¶](#unattended-installation "Permalink to this headline")

You may install MongoDB Community unattended on Windows from the command line
using msiexec.exe.
你可以在命令行里面使用msiexec.exe来安装MongoDB社区版。

1

#### 打开管理员命令行提示符[¶](#open-an-administrator-command-prompt "Permalink to this headline")

Press the Win key, type cmd.exe, and press Ctrl + Shift + Enter
to run the _Command Prompt_ as Administrator.
按下Win键，输入cmd.exe，接着按下Ctrl + Shift + Enter从而作为管理员运行命令行提示符。

Execute the remaining steps from the Administrator command prompt.
在管理员命令行提示符里面执行其它的步骤。

2

#### 对于Windows安装MongoDB[¶](#id2 "Permalink to this headline")

Change to the directory containing the .msi installation binary of your
choice and invoke:
切换到包含.msi安装二进制文件的文件夹并且调用：
```
msiexec.exe /q /i mongodb-win32-x86_64-2008plus-ssl-3.4.1-signed.msi ^
            INSTALLLOCATION="C:\Program Files\MongoDB\Server\3.4.1\" ^
            ADDLOCAL="all"

```

You can specify the installation location for the executable by
modifying the INSTALLLOCATION value.
你可以通过过修改INSTALLOCATION值来制定安装位置。

By default, this method installs all MongoDB binaries. To install specific
MongoDB component sets, you can specify them in the ADDLOCAL argument
using a comma-separated list including one or more of the following
component sets:
默认这个方法会安装MongoDB所有的二进制文件。为了安装特定的MongoDB组件集合，你可以制定他们通过添加ADDLOCAL参数，参数的格式是分号隔开的序列，这个序列包含一个或者多个下列的组件集合:

**组件集合**
**二进制文件**

Server
mongod.exe

Router
mongos.exe

Client
mongo.exe

MonitoringTools
mongostat.exe, mongotop.exe

ImportExportTools
mongodump.exe, mongorestore.exe, mongoexport.exe, mongoimport.exe

MiscellaneousTools
bsondump.exe, mongofiles.exe, mongooplog.exe, mongoperf.exe

For instance, to install _only_ the MongoDB utilities, invoke:
比如，如果只想安装MongoDB工具，则调用：

```
msiexec.exe /q /i mongodb-win32-x86_64-2008plus-ssl-3.4.1-signed.msi ^
            INSTALLLOCATION="C:\Program Files\MongoDB\Server\3.4.1\" ^
            ADDLOCAL="MonitoringTools,ImportExportTools,MiscellaneousTools"

```

## 运行MongoDB社区版[¶](#run-mongodb-community-edition "Permalink to this headline")

Warning
警告

Do not make [mongod.exe](../../reference/program/mongod.exe/#bin.mongod.exe) visible on public networks without
running in “Secure Mode” with the auth setting. MongoDB is
designed to be run in trusted environments, and the database does not
enable “Secure Mode” by default.
不要在没有认证设置里面没有设置“安全模式”的情况下让[mongod.exe](../../reference/program/mongod.exe/#bin.mongod.exe)在外网可见。MongoDB是被设计用于在可信任的环境中运行，数据不会默认开启“安全模式”。

1

### 设置MongoDB环境[¶](#set-up-the-mongodb-environment "Permalink to this headline")

MongoDB requires a [_data directory_](../../reference/glossary/#term-dbpath) to store all
data. MongoDB’s default data directory path is the absolute path
\data\db on the drive from which you start MongoDB. Create
this folder by running the following command in a
_Command Prompt_:
MongoDB需要一个[数据目录](../../reference/glossary/#term-dbpath)来存储所有的数据。MongoDB的默认数据目录路径是你启动MongoDB磁盘里面的绝对路径\data\db。通过在命令行提示符运行下面的命令来创建这个文件夹：

```
md \data\db

```

You can specify an alternate path for data files using the
_--dbpath_ option to
[mongod.exe](../../reference/program/mongod.exe/#bin.mongod.exe), for example:
你也可以使用--dbpath选项制定数据文件的路径，比如：

```
"C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe" --dbpath d:\test\mongodb\data

```

If your path includes spaces, enclose the entire path in double
quotes, for example:
如果你的路径包含空格，则应该将整个路径放在双引号里面，比如：

```
"C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe" --dbpath "d:\test\mongo db data"

```

You may also specify the dbpath in a [_configuration file_](../../reference/configuration-options/).
你也可以通过[配置文件](../../reference/configuration-options/)来制定dbpath。

2

### 启动MongoDB[¶](#start-mongodb "Permalink to this headline")

To start MongoDB, run [mongod.exe](../../reference/program/mongod.exe/#bin.mongod.exe). For example, from the
_Command Prompt_:
为了启动MongoDB,运行 [mongod.exe](../../reference/program/mongod.exe/#bin.mongod.exe)。比如在命令行提示符里：

```
"C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe"

```

This starts the main MongoDB database process. The waiting for
connections message in the console output indicates that the
[mongod.exe](../../reference/program/mongod.exe/#bin.mongod.exe) process is running successfully.
这启动了主要的MongoDB数据库进程。稍后出现的连接信息表明[mongod.exe](../../reference/program/mongod.exe/#bin.mongod.exe) 进程已经成功运行。

Depending on the security level of your system, Windows may pop up a
_Security Alert_ dialog box about blocking “some features” of
C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe from communicating
on networks. All users should select Private Networks, such as my home or
work network and click Allow access. For additional information on
security and MongoDB, please see the [_Security Documentation_](../../security/).
取决于你系统的安全等级，对于C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe与网络通信时候的“某些特性”，系统会弹出安全警告对话框。所有的用户应该选择专有网络，比如我家或者工作网络，并且点击允许。对于安全以及MongoDB的其他信息，可以参考[安全文档](../../security/)

3

### 连接MongoDB[¶](#connect-to-mongodb "Permalink to this headline")

To connect to MongoDB through the [mongo.exe](../../reference/program/mongo/#bin.mongo "mongo") shell,
open another _Command Prompt_.
通过[mongo.exe](../../reference/program/mongo/#bin.mongo "mongo") shell来连接MongoDB,打开另外一个命令行提示符。

```
"C:\Program Files\MongoDB\Server\3.4\bin\mongo.exe

```

If you want to develop applications using .NET, see the documentation
of [C# and MongoDB](https://docs.mongodb.com/ecosystem/drivers/csharp) for more information.
如果你想开发使用.NET的应用，参考文档[C# and MongoDB](https://docs.mongodb.com/ecosystem/drivers/csharp)来获取更多信息。

4

### 开始使用MongoDB[¶](#begin-using-mongodb "Permalink to this headline")

To help you start using MongoDB, MongoDB provides [_Getting
Started Guides_](../../#getting-started) in various driver editions. See
[_Getting Started_](../../#getting-started) for the available editions.
为了帮助你开始使用MongoDB,MongoDB在各个版本提供了[入门指导](../../#getting-started)。在 [_Getting Started_](../../#getting-started) 里面可以看到现有的版本。

Before deploying MongoDB in a production environment, consider the
[_Production Notes_](../../administration/production-notes/) document.
在将MongoDB部署到产品环境之前，阅读[_Production Notes_](../../administration/production-notes/)文档。

Later, to stop MongoDB, press Control+C in the terminal where the
[mongod](../../reference/program/mongod/#bin.mongod "mongod") instance is running.
之后，想暂停MongoDB的话，在运行[mongod](../../reference/program/mongod/#bin.mongod "mongod")实例的终端里面按Control+C。

## 为MongoDB社区版配置Windows服务[¶](#configure-a-windows-service-for-mongodb-community-edition "Permalink to this headline")

1

### 打开管理员命令行提示符[¶](#id3 "Permalink to this headline")

Press the Win key, type cmd.exe, and press Ctrl + Shift + Enter
to run the _Command Prompt_ as Administrator.
按下Win键，输入cmd.exe，接着按下Ctrl + Shift + Enter来运行以管理员身份运行明航提示符。

Execute the remaining steps from the Administrator command prompt.
在管理员命令行提示符里面运行以下步骤。

2

### 创建目录[¶](#create-directories "Permalink to this headline")

Create directories for your database and log files:
为你的数据库和日志文件创立文件夹：

```
mkdir c:\data\db
mkdir c:\data\log

```

3

### 创建配置文件[¶](#create-a-configuration-file "Permalink to this headline")

Create a configuration file. The file **must** set [systemLog.path](../../reference/configuration-options/#systemLog.path "systemLog.path").
Include additional
[_configuration options_](../../reference/configuration-options/) as appropriate.
创建配置文件。这个文件**必须**设置[systemLog.path](../../reference/configuration-options/#systemLog.path "systemLog.path")，也包含其他的[_configuration options_](../../reference/configuration-options/)。

For example, create a file at C:\Program Files\MongoDB\Server\3.4\mongod.cfg that specifies both
[systemLog.path](../../reference/configuration-options/#systemLog.path "systemLog.path") and [storage.dbPath](../../reference/configuration-options/#storage.dbPath "storage.dbPath"):
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

Important
重要

Run all of the following commands in _Command Prompt_ with
“Administrative Privileges”.
以“管理员权限”在命令行提示符里面运行下面所有的命令：

Install the MongoDB service by starting [mongod.exe](../../reference/program/mongod.exe/#bin.mongod.exe)
with the --install option and the -config
option to specify the previously created configuration file.
通过以--install选项启动[mongod.exe](../../reference/program/mongod.exe/#bin.mongod.exe)来安装MongoDB服务，并且通过-config选项来指定之前创建的配置文件。

```
"C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe" --config "C:\Program Files\MongoDB\Server\3.4\mongod.cfg" --install

```

To use an alternate dbpath, specify the path in the
configuration file (e.g. C:\mongodb\mongod.cfg) or
on the command line with the _--dbpath_ option.
为了指定dbpath，你可已在配置文件里面设置路径(e.g. C:\mongodb\mongod.cfg)或者在命令行立案通过--dbpath选项来设置。

If needed, you can install services for multiple instances of
[mongod.exe](../../reference/program/mongod.exe/#bin.mongod.exe) or [mongos.exe](../../reference/program/mongos.exe/#bin.mongos.exe). Install each service
with a unique _--serviceName_ and
_--serviceDisplayName_. Use
multiple instances only when sufficient system resources exist and your
system design requires it.
如果需要的话，你可以对[mongod.exe](../../reference/program/mongod.exe/#bin.mongod.exe)或者[mongos.exe](../../reference/program/mongos.exe/#bin.mongos.exe)的多个实例安装服务。对于每一个服务，设置一个特别的--serviceName以及--serviceDisplayName。只有在你系统资源允许并且你的系统设计需要这样的情况下才使用多个实例。

5

### 启动MongoDB服务[¶](#start-the-mongodb-service "Permalink to this headline")

```
net start MongoDB

```

6

### 停止或者移除MongoDB服务[¶](#stop-or-remove-the-mongodb-service-as-needed "Permalink to this headline")

To stop the MongoDB service use the following command:
使用下面的命令停止MongoDB服务：

```
net stop MongoDB

```

To remove the MongoDB service use the following command:
使用下面的命令移除MongoDB服务：

```
"C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe" --remove

```

## 为MongoDB社区版手动创建Windows服务[¶](#manually-create-a-windows-service-for-mongodb-community-edition "Permalink to this headline")

You can set up the MongoDB server as a _Windows Service_ that
starts automatically at boot time.
你可以将MongoDB服务器设置为Windows服务令其在启动的时候自动运行。

The following procedure assumes you have installed MongoDB Community using the
.msi installer with the path C:\Program Files\MongoDB\Server\3.2.
下面的流程是基于假设你已经通过.msi安装器在路径 C:\Program Files\MongoDB\Server\3.2里安装了MongoDB社区版。

If you have installed in an alternative directory, you will need to
adjust the paths as appropriate.
如果你已经在其他的目录安装过了，你可能需要调整路径。

1

### 打开管理员命令行提示符[¶](#id5 "Permalink to this headline")

Press the Win key, type cmd.exe, and press Ctrl + Shift + Enter
to run the _Command Prompt_ as Administrator.
按下Win键，输入cmd.exe，接着按下Ctrl + Shift + Enter来运行以管理员身份运行明航提示符。

Execute the remaining steps from the Administrator command prompt.
在管理员命令行提示符里面运行以下步骤。

2

### 创建文件夹[¶](#id7 "Permalink to this headline")

Create directories for your database and log files:

```
mkdir c:\data\db
mkdir c:\data\log

```

3

### Create a configuration file.[¶](#id9 "Permalink to this headline")

Create a configuration file. The file **must** set [systemLog.path](../../reference/configuration-options/#systemLog.path "systemLog.path").
Include additional
[_configuration options_](../../reference/configuration-options/) as appropriate.

For example, create a file at C:\Program Files\MongoDB\Server\3.4\mongod.cfg that specifies both
[systemLog.path](../../reference/configuration-options/#systemLog.path "systemLog.path") and [storage.dbPath](../../reference/configuration-options/#storage.dbPath "storage.dbPath"):

```
systemLog:
    destination: file
    path: c:\data\log\mongod.log
storage:
    dbPath: c:\data\db

```

4

### Create the MongoDB service.[¶](#create-the-mongodb-service "Permalink to this headline")

Create the MongoDB service.

```
sc.exe create MongoDB binPath= "\"C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe\" --service --config=\"C:\Program Files\MongoDB\Server\3.4\mongod.cfg\"" DisplayName= "MongoDB" start= "auto"

```

sc.exe requires a space between “=” and the configuration values
(eg “binPath= ”), and a “\” to escape double quotes.

If successfully created, the following log message will display:

```
[SC] CreateService SUCCESS

```

5

### Start the MongoDB service.[¶](#id11 "Permalink to this headline")

```
net start MongoDB

```

6

### Stop or remove the MongoDB service as needed.[¶](#id13 "Permalink to this headline")

To stop the MongoDB service, use the following command:

```
net stop MongoDB

```

To remove the MongoDB service, first stop the service and then
run the following command:

```
sc.exe delete MongoDB

```

## 其它资源[¶](#additional-resources "Permalink to this headline")

*   [MongoDB for Developers Free Course](https://university.mongodb.com/courses/M101P/about?jmp=docs)

*   [MongoDB for .NET Developers Free Online Course](https://university.mongodb.com/courses/M101N/about?jmp=docs)

*   [MongoDB Architecture Guide](https://www.mongodb.com/lp/white-paper/architecture-guide?jmp=docs)

[](https://twitter.com/MongoDB)
[](https://www.youtube.com/user/MongoDB)
[](https://www.facebook.com/mongodb)
[](https://plus.google.com/u/1/101024085748034940765/posts?cfem=1)
​                