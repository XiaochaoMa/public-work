# Redfish 验证器

使用说明及代码下载地址：
https://github.com/DMTF/Redfish-Service-Validator



Redfish服务验证器是python3工具，用于根据Redfish CSDL模式检查具有Redfish服务接口的任何“设备”的符合性。 该工具被设计为与设备无关，并且是基于Redfish规范和打算由设备支持的架构来驱动的。

Redfish服务验证器基于Python 3，并且必须先在客户端系统上安装Python框架，然后才能在系统上安装和执行该工具。 此外，需要安装以下软件包，并可以从python环境中访问以下软件包：

beautifulsoup4 ---- https://pypi.python.org/pypi/beautifulsoup4

requests ---- https://github.com/kennethreitz/requests 
(Documentation is available at http://docs.python-requests.org/)

lxml ---- https://pypi.python.org/pypi/lxml

可以通过运行以下命令来安装以上必备组件：
pip3 install -r requirements.txt

没有基于Windows或Linux OS的依赖关系。 结果日志以HTML格式生成，并且需要适当的浏览器（Chrome，Firefox，IE等）才能在客户端系统上查看日志。

## 安装--->

将RedfishServiceValidator.py 放入所需的工具根目录。 在工具根目录中创建以下子目录：“ config”，“ logs”，“ SchemaFiles”。 将示例config.ini文件放置在“ config”目录中。 将工具要使用的CSDL模式文件{CSDL (Conceptual Schema Definition Language) 。主要是概念层，实体类，实体类集合，实体属性的描述。开发人员直接使用概念层定义的实体类和属性。内部有包含数据表和关联的子节点}放置在模式目录的根目录或config.ini中指定的目录中。

## 执行步骤--->

Redfish Interop验证程序设计为作为纯命令行界面工具执行，在工具执行过程中不需要任何中间输入。 但是，该工具需要有关系统详细信息，DMTF模式文件等的各种输入，在执行过程中该工具会消耗这些输入以生成一致性报告日志。 以下是有关设置工具以在任何已标识的Redfish设备上执行以进行一致性测试的逐步说明：
修改config \ config.ini文件以在以下部分下输入系统详细信息

[系统信息]

TargetIP = <被测系统的IPv4地址>

SystemInfo = <描述系统>

UserName = <系统上管理员的用户标识>

密码= <管理员密码>

ForceAuth = <在不安全的通讯上强制身份验证>

AuthType = <上述凭据的授权类型（无，基本，会话）>

UseSSL = <是/否>

CertificateCheck = <是/否>

CertificateBundle = ca_bundle指定包含受信任CA证书的捆绑包（文件或目录）。 有关创建捆绑软件的提示，请参见SelfSignedCerts.md。

该工具具有一个选项，如果客户端系统上未安装证书，则可以忽略SSL证书检查。 可以使用config.ini文件的以下参数打开或关闭证书检查。 默认情况下，该参数设置为False。 UseSSL确定是否使用https协议。 如果为False，则还将禁用认证。

[选项]--options

OemCheck = <是/否>指定是否要检查OEM属性

UriCheck = <True / False>指定是否要将资源URI与Redfish.Uris比较

VersionCheck =（字符串）指定版本以测试服务，切换特定协议的配置（自动版本留空）

LocalOnlyMode-（布尔值）仅针对置于MetadataFilePath根目录中的Schema测试属性。

ServiceMode-（布尔值）仅针对服务上存在的资源/架构测试属性

PreferOnline-（布尔值）优先于本地模式的联机/服务模式。这还将禁用将所有架构下载到指定的元数据目录

MetadataFilePath-（字符串）此属性指向DMTF模式文件位置的位置，该位置由xml文件填充

Schema_Pack-（字符串）DMTF Schema官方压缩包的URL路径，将在用户的本地模式目录中提取。使用“最新”来拉最新的邮政编码。 （与选项LocalOnly一起使用）

Timeout -（整数）超时请求前的时间间隔

SchemaSuffix-（字符串）搜索本地硬盘驱动器架构时，如果无法从服务的元数据中获取期望的xml，请附加此内容

HttpProxy-（URL）对外部URL的HTTP请求的代理（例如：HttpProxy = http://192.168.1.1:8888）

HttpsProxy-（HTTPS）请求到外部URL的（URL）代理（例如：HttpsProxy = http://192.168.1.1:8888）

注意：HttpProxy / HttpsProxy不适用于对被测系统的请求，仅适用于系统外部的URL。

其他选项可用于缓存文件，链接限制，采样和目标有效负载：

CacheMode = [Off，Prefer，Fallback]-使用缓存的选项，它将允许用户在对服务进行资源调用期间覆盖或回退到磁盘上的文件，用本地有效负载替换json响应

CacheFilePath =缓存目录的路径。 默认为“ ./cache”。 如果您需要在特定有效负载的json上测试对服务响应的微小更改，请考虑使用CacheMode

LinkLimit = TypeName：##-选项以限制从集合接受的链接数量，默认为LogEntry：20

sample =（整数）要验证的大型集合中的随机成员数。 默认值为验证所有成员。 如果指定的值为零或负数，则将验证所有成员。 如果LinkLimit和Sample应用于给定的集合，则LinkLimit优先。

[validator]

LogPath-（字符串）存储生成器日志的路径

PayloadMode = [Default，Tree，Single，TreeFile，SingleFile]-验证目标的选项，允许指定文件或特定的URI和遍历行为

PayloadFilePath = URI /文件的路径

在为测试中的系统更新了以上详细信息之后，可以通过键入以下命令并使用详细选项，从命令提示符中触发Redfish服务验证程序：
python3 RedfishServiceValidator.py -c config/config.ini (-v) (--verbose_checks) (--debug_logging)

另外，所有这些选项都可通过命令行使用。 您可以选择通过命令行覆盖一些配置条目。要查看这些选项，请运行以下命令：
python3 RedfishServiceValidator.py -h (-v) (--verbose_checks) (--debug_logging)

为了在没有配置文件的情况下运行，必须指定选项--ip。
python3 RedfishServiceValidator.py --ip host:port [...]

还可以使用GUI来让用户使用表单填写配置选项，然后按下按钮以执行测试。 以下命令将启动该工具的GUI
python3 RedfishServiceValidatorGui.py

要运行单元测试，请使用以下命令：
python3 -m unittest tests/*.py

为了将csv报告打印到文本日志中，可以将--csv_report命令添加到命令行中。 要将以前的HTML日志转换为csv文件，请使用以下命令：

python3 tohtml.py htmllogfile

## 执行流程--->

一.Redfish服务验证器通过使用服务根URI查询服务并获取所有设备信息，支持的资源及其链接，从服务根资源架构开始。 一旦根据其模式验证了服务根查询的响应，该工具就会遍历该服务返回的所有集合和导航属性。
①从元数据中，收集服务可能指定的所有XML，并将它们存储在工具指定的目录中
二.对于返回的每个导航属性/资源集合，它执行以下操作：
①从相应的资源收集架构文件中读取所有资源的导航/收集。
②读取与特定资源相关的架构文件，从资源架构文件收集有关各个属性的所有信息，并将它们存储到字典中。
③使用单个资源uri查询服务，并使用GET方法针对从架构文件收集的属性来验证返回的所有属性，以确保支持所有强制属性。
三.重复步骤2，直到覆盖所有URI和资源为止。


验证资源后，可能会进行以下类型的测试：
--到达任何资源后，使用正则表达式验证其有效负载内的@odata条目。 如果失败，则返回“ failPayloadError”。
--尝试启动Resource对象，该对象需要有效的JSON有效负载的@ odata.type和Schema，否则返回“ problemResource”或“ exceptionResource”并终止，否则返回“ passGet”
--启动资源后，开始根据其架构定义（无注释，其他支持，区分大小写）来验证每个可用的属性：
一.首先检查属性是否可以为空或为必需，然后根据其要求或可空性进行传递。
二.对于集合，请验证其内部的每个属性，并期望使用列表而不是单个属性，否则请正常进行验证：
  ①对于基本类型，例如Int，Bool，DateTime，GUID，String等，请检查适当的类型，正则表达式和范围。
  ②对于复杂类型，请验证复杂内部的每个属性，包括注释和其他属性。
  ③对于枚举类型，请检查并在架构定义枚举类型（区分大小写）中查看给定值。
  ④对于实体类型，请通过执行GET检查@ odata.id给出的链接是否通过正确的类型将客户端发送到适当的资源。
--完成后，使用有效载荷中所有可用的注释执行相同的例程。
--如果有效负载中存在任何未经验证的条目，请确定其他属性对于此资源是否合法，否则引发“ failAdditional”错误。

## 符合性日志-摘要和详细的符合性报告--->

Redfish服务验证器在“ logs”文件夹下生成一个html报告，名为ConformanceHtmlLog_MM_DD_YYYY_HHMMSS.html。该报告提供了检查的各个属性的详细视图，以及每个检查是否符合要求的通过/失败/跳过/警告状态。

此外，当HTML日志不足时，可以引用一个详细的日志文件来诊断工具或架构问题。

## 测试状态--->

每个GET操作的测试结果将报告如下：
PASS：如果操作成功并返回成功代码（例如200、204）
Fail：如果操作由于GET方法执行或某些配置中提到的原因而失败。
SKIP：如果服务不支持检查的属性或方法不是必需的。

## 局限性--->

Redfish服务验证程序涵盖了服务上的所有GET执行。 以下是不在此范围内的某些要点。
由于依赖于服务的内部因素，补丁/帖子/跳过/顶部/标题未作为Redfish服务验证程序的一部分。
Redfish服务验证程序不涵盖一次测试多个服务。 要执行此操作，我们必须通过单独运行它来重新运行该工具。
工具不支持使用复杂的$ entity路径的@ odata.context

## 构建独立的Windows可执行文件--->

pyinstaller模块用于将环境打包为独立的可执行文件。 可以使用以下命令进行安装：
pip3 install pyinstaller
在Windows系统中，以下命令可用于构建Windows可执行文件RedfishServiceValidator.exe，该文件可在dist文件夹中找到：
pyinstaller -F -w -i redfish.ico -n RedfishServiceValidator.exe RedfishServiceValidatorGui.py

## 发布过程--->

1使用上一版本以来的更改列表更新CHANGELOG.md
2更新RedfishServiceValidator.py中的tool_version变量以反映新的工具版本
3将更改推送到Github
4如上节所述创建独立的可执行文件
5为该发行版创建一个zip文件，其中包含生成的exe文件和README.md
  5.1名称格式：Redfish-Service-Validator-X.Y.Z-Windows.zip
6在Github中创建新版本并附加zip文件

--



--Xiaochao Ma