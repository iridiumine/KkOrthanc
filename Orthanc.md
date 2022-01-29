# 一些关于Orthanc学习使用中的记录

## 何为Orthanc？

orthanc是一款能够提供RESTful API的迷你影像服务器，orthanc提供了数据库存储dicom索引的插件（Postgresql，Mysql，SQLlite）。

- 关于 RESTful API

  https://www.runoob.com/w3cnote/restful-architecture.html

  REST全称是Representational State Transfer，即表征性状态转移。它首次出现在2000年Roy Fiedling（HTTP规范的主要编写者之一）的博士论文中，他在论文中提到：“我这篇文章的写作目的，就是想在符合架构原理的前提下，理解和评估以网络为基础的应用软件的架构设计，得到一个功能强、性能好、适宜通信的架构。REST指的是一组架构约束条件和原则。”如果一个架构符合REST的约束条件和原则，我们就称它为RESTful架构。

  REST本身没有创造新的技术、组件或服务，而隐藏在RESTful背后的理念就是使用Web的现有特征和能力，更好地使用现有Web标准的中的一些准则和约束。虽然REST本身受Web技术的影响很深，但是理论上REST架构风格并不是绑定在HTTP上，只不过目前HTTP是唯一与REST相关的实例。所以我们这里描述的REST也是通过HTTP实现的REST。

  https://www.jianshu.com/p/dbee5199cf23

  REST，简单来说，就是用URI（Uniform Resource Identifier）表示资源，用HTTP方法（GET，POST，PUT，DELETE）表征对这些资源的操作。

  RESTful API就是REST风格的API，它要求前端以一种预定义的语法格式发送请求。

- 关于RESTful架构遵循的统一接口原则

  接口应该使用标准的HTTP方法如：GET，PUT，POST和DELETE等，考虑安全性和幂等性（多次操作，结果一致），但并不阻止扩展带特殊语义的方法，只要方法对资源的操作有着具体的、可识别的语义即可，并能够保持整个接口的统一性。

- 关于REST中的状态转移

  REST原则中包含无状态通信原则（指服务端不应该保存客户端状态）。实际上，状态应该区分应用状态和资源状态，客户端负责维护应用状态，而服务端维护资源状态。客户端与服务端端交互必须是无状态的，并在每一次请求中包含处理该请求所需的一切信息。服务端不需要在请求间保留应用状态，只有在接收到实际请求的时候，服务端才会关注应用状态。这种无状态通信原则，使得服务端和中介能够理解独立的请求和响应。在多次请求中，同一客户端也不再需要依赖于同一服务器，方便实现高可扩展和高可用性的服务端。“会话”状态不是作为资源状态保存在服务端的，而是被客户端作为应用状态进行跟踪的。客户端应用状态在服务端提供的超媒体的指引下发生变迁。服务端通过超媒体告诉客户端当前状态有哪些后续状态可以进入。

- 关于dicom

  DICOM（Digital Imaging and Communications in Medicine），即医学数字成像和通信，是医学图像和相关信息的国际标准（ISO 12052），它定义了质量能满足临床需要的可用于数据交换的医学图像格式。（扩展名：.dcm）

## 关于docker

https://baike.baidu.com/item/Docker/13344470

Docker是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的镜像中，然后发布到任何流行的Linux或Windows操作系统的机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。

一个完整的Docker有以下几个部分组成：

- Docker Client客户端
- Docker Daemon守护进程
- Docker Image镜像
- Docker Container容器

### Docker架构

Docker使用客户端-服务器架构模式，使用远程API来管理和创建Docker容器。Docker容器通过Docker镜像来创建。容器与镜像的关系类似于面向对象编程中的对象与类。

Docker采用C/S架构，Docker daemon作为服务端接受来自客户的请求，并处理这些请求（创建、运行、分发容器）。客户端和服务端既可以运行在一个机器上，也可通过socket或者RESTful API来进行通信。

Docker daemon一般在宿主主机后台运行，等待接收来自客户端的消息。Docker客户端则为用户提供一系列可执行命令，用户用这些命令实现跟Docker daemon交互。

### 关于docker和orthanc的浅薄认识

orthanc是一个服务器（服务端），它如果想要运行并提供服务（客户端是浏览器），则需要有相应的资源。docker作为一个C/S架构模式的应用容器引擎，通过orthanc的镜像来创建orthanc容器，相当于利用docker分配了本地资源给orthanc服务器，并且依靠docker管理相应的资源（因为在docker-orthanc这层关系中，实际的资源是由docker掌握，orthanc服务器只是作为客户请求资源）。之所以我在本地上用浏览器（也就是客户端）访问http://localhost:8042/之后点击Lookup却不显示任何信息，是因为我没有在docker管理的本地orthanc服务器上上传过任何资源。由于orthanc能提供RESTful API，而REST是通过HTTP实现的，那么本次的任务是在orthanc服务端，在遵循RESTful架构的统一接口原则的前提下，使用标准的HTTP方法，尝试扩展带特殊语义的方法（例如直接调出病人的uid一类的）。

首先，我们需要做到资源和容器的分离，所以相当于我们运行orthanc容器的时候，我们的资源不需要再镜像中本来就有，所以我们可以以挂载的方式，将我们需要的数据文件夹挂载到容器的目录中，然后通过修改配置文件中orthanc的数据库路径来实现数据storage的设置（docker本身就是使用了linux的文件管理方式）。——李浩学长

### 关于在docker中启动orthanc

docker启动orthanc，默认端口：8042

```
$ docker run -p 4242:4242 -p 8042:8042 --rm jodogne/orthanc
```

修改配置文件

```
$ docker run --rm --entrypoint=cat jodogne/orthanc /etc/orthanc/orthanc.json > /tmp/orthanc.json
```

使用更新的配置文件启动Orthanc

```
$ docker run -p 4242:4242 -p 8042:8042 --rm -v /tmp/orthanc.json:/etc/orthanc/orthanc.json:ro jodogne/orthanc
```

## 关于Orthanc接口

Orthanc的主要优势在于其内置的RESTful API，该API可用于从外部应用程序驱动orthanc，并且与开发应用的编程语言无关。重要的是，Orthanc Explorer（一个Orthanc嵌入式管理接口）完全依赖于上述RESTful API来实现它的所有功能。

Note：所有示例都使用cURL命令行工具进行说明，但等效调用可以很容易地转换为任何同时支持HTTP和JSON的编程语言（比如python）。（cURL：一个利用URL语法在命令行下工作的文件传输工具，支持文件上传和下载）

### 发送DICOM图片

```
$ curl -X POST http://localhost:8042/instances --data-binary @CT.X.1.2.276.0.7230010.dcm
```

orthanc将使用带有存储实例位置的JSON文件作为回应，例如

```json
{
  "ID" : "e87da270-c52b-4f2a-b8c6-bae25928d0b0",
  "Path" : "/instances/e87da270-c52b-4f2a-b8c6-bae25928d0b0",
  "Status" : "Success"
}
```

Orthanc的代码分发包含了一个使用REST API来递归地上传某些文件中的内容到Orhtanc的示例python脚本。（https://hg.orthanc-server.com/orthanc/file/Orthanc-1.9.7/OrthancServer/Resources/Samples/ImportDicomFiles/ImportDicomFiles.py）

```python
python ImportDicomFiles.py localhost 8042 ~/DICOM/
```

从Orthanc1.8.1开始，Orthanc的源分发包含了另一个python脚本OrthancImport.py，该脚本相比ImportDicomFiles.py提供了更多的功能。它可以显著导入.zip、.tar.gz或.tar.bz2档案的内容，而无需先解压缩它们。它还提供了更全面的命令行选项。（https://hg.orthanc-server.com/orthanc/file/Orthanc-1.9.7/OrthancServer/Resources/Samples/ImportDicomFiles/OrthancImport.py）

如果使用Powershell (>= 3.0)，可以通过以下内容向Orthanc发送单个Dicom实例：

```powershell
# disabling the progress bar makes the Invoke-RestMethod call MUCH faster
$ProgressPreference = 'SilentlyContinue'

# upload it to Orthanc
$reply = Invoke-RestMethod http://localhost:8042/instances -Method POST -InFile CT.X.1.2.276.0.7230010.dcm

# display the result
Write-Host "The instance can be retrieved at http://localhost:8042$($reply.Path)"
```

### 访问Orthanc内容

Orthanc使用“患者，研究，系列，实例”的DICOM标准模型来组织存储的DICOM资源。每个DICOM资源都与唯一的标识符相关联。

#### 列出所有的DICOM资源

以下是如何列出存储在本地Orthanc实例中的所有DICOM资源的方法：

```
$ curl http://localhost:8042/patients
$ curl http://localhost:8042/studies
$ curl http://localhost:8042/series
$ curl http://localhost:8042/instances
```

请注意，此命令的结果是一个JSON文件，其中包含资源标识符数组。

#### 访问一个患者

要访问单个资源，将其标识符添加到URI中。例如，通过如下示例检索一位患者的主要信息：

```
$ curl localhost:8042/patients/8c4d0f60-4c7b0273-785ba198-aded6ba3-a47a89a7
```

Orthanc回应的JSON文件：

```json
{
   "ID" : "8c4d0f60-4c7b0273-785ba198-aded6ba3-a47a89a7",
   "IsStable" : true,
   "LastUpdate" : "20220120T074119",
   "MainDicomTags" : {
      "PatientBirthDate" : "20210223",
      "PatientID" : "#00385",
      "PatientName" : "急诊",
      "PatientSex" : "Male"
   },
   "Studies" : [ "2b15e950-33980cf9-cde6a7a4-acfbf97d-3b31639c" ],
   "Type" : "Patient"
}
```

#### 从患者向下浏览到实例

Studies信息域列出了与患者相关的所有DICOM研究。因此，考虑到上面的病人，按照以下方式进入她的DICOM等级体系：

```
$ curl http://localhost:8042/studies/9ad2b0da-a406c43c-6e0df76d-1204b86f-78d12c15
```

Orthanc回应的JSON文件：

```json
{
   "ID" : "2b15e950-33980cf9-cde6a7a4-acfbf97d-3b31639c",
   "IsStable" : true,
   "LastUpdate" : "20220120T074119",
   "MainDicomTags" : {
      "AccessionNumber" : "",
      "InstitutionName" : "BMEC RFLab of USTC",
      "ReferringPhysicianName" : "magforce",
      "StudyDate" : "20210223",
      "StudyDescription" : "哈哈哈哈，你好啊",
      "StudyID" : "#00385",
      "StudyInstanceUID" : "1.2.276.0.7230010.3.1.2.1824580983.36480.1628595363.853",
      "StudyTime" : "151712"
   },
   "ParentPatient" : "8c4d0f60-4c7b0273-785ba198-aded6ba3-a47a89a7",
   "PatientMainDicomTags" : {
      "PatientBirthDate" : "20210223",
      "PatientID" : "#00385",
      "PatientName" : "急诊",
      "PatientSex" : "Male"
   },
   "Series" : [ "9b9a96f5-f8d4a65c-9666d5b0-cc5eba6b-d84abd95" ],
   "Type" : "Study"
}
```

MainDicomTags标签现在是那些与study层级相关的标签。可以在ParentPatient字段中检索患者的标识符，该标识符可用于向上移动DICOM层次结构。但我们宁愿使用series数组进入series层级。下一个命令将返回有关刚刚报告的series信息：

```
$ curl localhost:8042/series/9b9a96f5-f8d4a65c-9666d5b0-cc5eba6b-d84abd95
```

Orthanc回应的JSON文件：

```json
{
   "ExpectedNumberOfInstances" : null,
   "ID" : "9b9a96f5-f8d4a65c-9666d5b0-cc5eba6b-d84abd95",
   "Instances" : [
      "5dbdad22-d21b8e67-83358204-fc591c97-55571dda",
      "33cef45b-b3599f87-797a7187-b3500703-834c0260",
      "a7400f1c-ce35011a-e8dc5f8f-47e26419-ae09dad2",
      "a07c8988-12d751ff-a2f99af0-373e70e4-d4bf7d8a",
      "30b11298-e4a50e64-92060427-2f8f7744-1d1baef3",
      "a8ed5a60-bbca3aa2-0d143ce9-ede2e62f-feb7043a",
      "251719c8-0a57ee0d-1605da9d-1c660ba5-2a9b8dc9",
      "846d44b5-5eadd8a7-2e6eb133-e197e2af-4e98c4bd",
      "1a306711-4050498e-ace2bb12-b88cd82c-c431b779"
   ],
   "IsStable" : true,
   "LastUpdate" : "20220120T074119",
   "MainDicomTags" : {
      "BodyPartExamined" : "Head",
      "ImageOrientationPatient" : "1\\0\\0\\0\\1\\0",
      "Manufacturer" : "Fuqing Company",
      "Modality" : "MR",
      "OperatorsName" : "迈力,magforce,医疗科技",
      "ProtocolName" : "LocGRE",
      "SeriesDate" : "20210223",
      "SeriesDescription" : "This is my auto generated LocGRE sequence library.",
      "SeriesInstanceUID" : "1.2.276.0.7230010.3.1.3.1824580983.36480.1628595363.854",
      "SeriesNumber" : "",
      "SeriesTime" : "151723",
      "StationName" : "RFLab"
   },
   "ParentStudy" : "2b15e950-33980cf9-cde6a7a4-acfbf97d-3b31639c",
   "Status" : "Unknown",
   "Type" : "Series"
}
```

实例层级：

```
$ curl localhost:8042/instances/5dbdad22-d21b8e67-83358204-fc591c97-55571dda
```

Orthanc回应的JSON文件：

```json
{
   "FileSize" : 525970,
   "FileUuid" : "9f28a7cd-f080-48f1-8e70-93ded32e585b",
   "ID" : "5dbdad22-d21b8e67-83358204-fc591c97-55571dda",
   "IndexInSeries" : 3,
   "MainDicomTags" : {
      "ImageOrientationPatient" : "1\\0\\0\\0\\1\\0",
      "ImagePositionPatient" : "-135\\-134.4726562\\10",
      "InstanceCreationDate" : "20210223",
      "InstanceCreationTime" : "151723.439892",
      "InstanceNumber" : "3",
      "SOPInstanceUID" : "1.2.276.0.7230010.3.1.4.1824580983.36480.1628595363.863"
   },
   "ParentSeries" : "9b9a96f5-f8d4a65c-9666d5b0-cc5eba6b-d84abd95",
   "Type" : "Instance"
}
```

该实例的父系列中的索引为3。该实例存储为525970字节的原始DICOM文件。使用以下命令下载此DICOM文件：（下载路径为当前位置）

```
$ curl http://localhost:8042/instances/5dbdad22-d21b8e67-83358204-fc591c97-55571dda/file > Instance.dcm
```

## 关于如何写一个自己的库

https://zhuanlan.zhihu.com/p/288557764

1.下载一个发布库的官方模版

```
git clone https://github.com/navdeep-G/setup.py
```

2.修改模块名称和setup.py中的信息

3.在core.py中编写自己的代码

4.将库打包（在项目根目录下）

```
python setup.py sdist 
```

5.将库安装在本地（在dist目录下）

```
pip install Qtool-0.0.1.tar.gz
```

6.验证

在命令行中导入写的模块

```python
import xxx
help(xxx.xxx)
```

7.发布到PyPI官网上（https://pypi.org/manage/projects/）修改.pypirc文件，使用用户名和密码

```
open ~/.pypirc

[pypi]
  username = iridiumine
  password = 13109EFzsy_PyPI
```

8.将打包的tar.gz文件发布（在项目根目录下）（每次更新要更新版本号—setup.py）

```
python3 -m twine upload dist/* 
```

另：使用正确的pip安装

```
python3 -m pip install KkOrthanc
```

使用pip更新库

```
python3 -m pip install --upgrade KkOrthanc
```

## 关于上传到github库

https://blog.csdn.net/caseywei/article/details/81051946

1.新建文件夹

```
mkdir KkOrthanc
cd KkOrthanc
```

2.初始化

```
git init
git status
```

3.将本地文件复制进KkOrthanc

4.上传

```
git add .
git commit -m "xxx"
git remote add origin git@github.com:iridiumine/KkOrthanc.git
git push -f origin master
```

## Requests库

https://docs.python-requests.org/en/latest/

### 基于Apache开源协议

- 五种开源协议：BSD，Apache，GPL，LGPL，MIT

### HTTP的八种请求类型

- OPTIONS：返回服务器针对特定资源所支持的HTTP请求方法。也可以利用向Web服务器发送'*'的请求来测试服务器的功能性。 

- HEAD：向服务器索要与GET请求相一致的响应，只不过响应体将不会被返回。这一方法可以在不必传输整个响应内容的情况下，就可以获取包含在响应消息头中的元信息。 

- GET：向特定的资源发出请求。 

  - 请求结果常见API

    ```python
    response=requests.get(url)
    print(response.status_code)   #响应结果状态码：200(成功)
    print(response.text)          #响应结果文本内容
    print(response.content)       #响应结果二进制内容(处理图片等)
    print(response.url)           #响应结果的请求url
    print(response.json())        #响应结果转成json字符串
    print(response.encoding)      #响应结果编码
    ```

- POST：向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST请求可能会导致新的资源的创建和/或已有资源的修改。 

- PUT：向指定资源位置上传其最新内容。

- DELETE：请求服务器删除Request-URI所标识的资源。 

- TRACE：回显服务器收到的请求，主要用于测试或诊断。 

- CONNECT：HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器。

### JSON相应内容

- 如果 JSON 解码失败， r.json() 就会抛出一个异常。例如，响应内容是 401 (Unauthorized)，尝试访问 r.json() 将会抛出 ValueError: No JSON object could be decoded 异常。
- 需要注意的是，成功调用 r.json() 并不意味着响应的成功。有的服务器会在失败的响应中包含一个 JSON 对象（比如 HTTP 500 的错误细节）。这种 JSON 会被解码返回。要检查请求是否成功，请使用 r.raise_for_status() 或者检查 r.status_code 是否和你的期望相同。

### 相应状态码

- 检测响应状态码：

  ```python
  >>> r = requests.get('http://httpbin.org/get')
  >>> r.status_code
  200
  ```

- 为方便引用，Requests还附带了一个内置的状态码查询对象：

  ```python
  >>> r.status_code == requests.codes.ok
  True
  ```

- 如果发送了一个错误请求(一个 4XX 客户端错误，或者 5XX 服务器错误响应)，可以通过Response.raise_for_status() 来抛出异常：

  ```python
  >>> bad_r = requests.get('http://httpbin.org/status/404')
  >>> bad_r.status_code
  404
  
  >>> bad_r.raise_for_status()
  Traceback (most recent call last):
    File "requests/models.py", line 832, in raise_for_status
      raise http_error
  requests.exceptions.HTTPError: 404 Client Error
  ```

### 响应头

- 查看以一个 Python 字典形式展示的服务器响应头：

  ```python
  >>> r.headers
  {
      'content-encoding': 'gzip',
      'transfer-encoding': 'chunked',
      'connection': 'close',
      'server': 'nginx/1.0.4',
      'x-runtime': '148ms',
      'etag': '"e1ca502697e5c9317743dc078f67693f"',
      'content-type': 'application/json'
  }
  ```

- 使用任意形式来访问这些响应头字段：

  ```python
  >>> r.headers['Content-Type']
  'application/json'
  
  >>> r.headers.get('content-type')
  'application/json'
  ```

### Cookie

- 快速访问某个响应中可能包含的cookie：

  ```python
  >>> url = 'http://example.com/some/cookie/setting/url'
  >>> r = requests.get(url)
  
  >>> r.cookies['example_cookie_name']
  'example_cookie_value'
  ```

- 发送cookies到服务器：

  ```python
  >>> url = 'http://httpbin.org/cookies'
  >>> cookies = dict(cookies_are='working')
  
  >>> r = requests.get(url, cookies=cookies)
  >>> r.text
  '{"cookies": {"cookies_are": "working"}}'
  ```

- Cookie 的返回对象为 RequestsCookieJar，它的行为和字典类似，但接口更为完整，适合跨域名跨路径使用。还可以把 Cookie Jar 传到 Requests 中：

  ```python
  >>> jar = requests.cookies.RequestsCookieJar()
  >>> jar.set('tasty_cookie', 'yum', domain='httpbin.org', path='/cookies')
  >>> jar.set('gross_cookie', 'blech', domain='httpbin.org', path='/elsewhere')
  >>> url = 'http://httpbin.org/cookies'
  >>> r = requests.get(url, cookies=jar)
  >>> r.text
  '{"cookies": {"tasty_cookie": "yum"}}'
  ```

### 错误与异常

所有Requests显式抛出的异常都继承自 requests.exceptions.RequestException。

- 遇到网络问题（如：DNS 查询失败、拒绝连接等）时，Requests 会抛出一个 ConnectionError 异常。
- 如果 HTTP 请求返回了不成功的状态码， Response.raise_for_status() 会抛出一个 HTTPError 异常。
- 若请求超时，则抛出一个 Timeout 异常。
- 若请求超过了设定的最大重定向次数，则会抛出一个 TooManyRedirects 异常。

## 关于python模块化开发

https://blog.csdn.net/qq_42992919/article/details/96852814

### 什么是模块化开发

- 模块化是代码组成的一种方式，Python中的每一个文件就是一个模块
- 在Python中，文件有三种方式进行组织：Python文件（也就是Python模块）、目录（directory）、包（package）

### 如何进行模块化开发以及注意点

- 功能模块的分类
- 功能模块接口的定义
- 异常捕获

### python文件与普通文件的区别

python中的文件也就是以.py后缀名结尾的文件，每一个.py后缀名结尾的文件就是python的一个模块，能够被其他模块所导入

### 包和目录的区别

包内部多一个\_\_init\_\_.py文件，使得包能够被模块导入，而目录不能

### 相对导入和绝对导入

- 绝对导入：从python的搜索路径中导入对应的包中的模块
- 相对导入：必须在包内部使用，且有相对导入的模块，无法直接运行，必须使用python -m package.module这样的方式运行，或者被\_\_main\_\_模块导入，之后使用
- 类似绝对路径和相对路径的区别

### 使用all过滤模块属性的可见性

- 在模块导入中，模块的属性都能够被导入，没有私有和保护属性的概念
- 但是在使用from package import \*的时候，保护属性和私有属性不会被导入
- 使用\_\_all\_\_ = ['a', 'b']可以显示声明能够被导入的属性

### 关于if \_\_name\_\_ == '\_\_main\_\_：

- 用于测试，功能调试，不让测试代码污染模块
- if \_\_name__ == '\_\_main\_\_'相当于是Python模拟的程序入口。由于模块之间相互引用，不同模块可能都有这样的定义，而入口程序只能有一个，哪个入口程序被选中取决于 \_\_name\_\_的值，\_\_name\_\_是内置变量，用于表示当前模块的名字。如果一个模块被直接运行，其 \_\_name\_\_ 值为\_\_main\_\_，如果是其他模块导入该模块的话，该模块的\_\_name\_\_属性值为包名。

### 模块的内置属性

- \_\_doc__：模块中用于描述的文档字符串
- \_\_name__：模块名
- \_\_file__：模块保存的路径

### 模块设计的一般原则

- 先设计API，再实现模块
- 控制模块规模，只提供需要的函数，降低复杂性
- 在模块中编写测试代码
- 使用私有函数实现不被外部客户端调用的模块函数
- 通过文档获取模块帮助信息

## python面向对象

### 简介

- 类：用来描述具有**相同的属性和方法**的**对象**的集合。它定义了该集合中每个对象所共有的属性和方法。对象是类的实例。
- 类变量：类变量在整个实例化的对象中是公用的。类变量定义在类内且在函数体之外。类变量通常不作为实例变量使用（使用类的名称访问类变量）。
- 数据成员：类变量或者实例变量，用于处理类及其实例对象的相关的数据。
- 方法重写：如果从父类继承的方法不能满足子类的需求，可以对其进行改写，这个过程叫方法的覆盖（override），也称为方法的重写。
- 局部变量：定义在方法中的变量，只作用于当前实例的类。
- 实例变量：在类的声明中，属性是用变量来表示的。这种变量就称为实例变量，是在类声明的内部但是在类的其他成员方法之外声明的。
- 继承：即一个派生类（derived class）继承基类（base class）的字段和方法。继承也允许把一个派生类的对象作为一个基类对象对待。
- 实例化：创建一个类的实例，类的具体对象。
- 方法：类中定义的函数。
- 对象：通过类定义的数据结构实例。对象包括两个数据成员（类变量和实例变量）和方法。

### 一些知识

- \_\_init()\_\_方法是一种特殊的方法，被称为类的构造函数或初始化方法，当创建了这个类的实例时就会调用该方法。
- self 代表类的实例，self 在定义类的方法时是必须有的，虽然在调用时不必传入相应的参数。
- 使用.来访问对象的属性。
- 在调用基类的方法时，需要加上基类的类名前缀，且需要带上 self 参数变量。区别在于类中调用普通函数时并不需要带上 self 参数。
- \_\_private\_attrs：两个下划线开头，声明该属性为私有，不能在类的外部被使用或直接访问。在类内部的方法中使用时 self.__private_attrs。
- 在类的内部，使用 def 关键字可以为类定义一个方法，与一般函数定义不同，类方法必须包含参数 self,且为第一个参数。
- \_\_private\_method：两个下划线开头，声明该方法为私有方法，不能在类的外部调用。在类的内部调用 self.__private_methods
- 单下划线、双下划线、头尾双下划线说明：
  - \_\_foo\_\_: 定义的是特殊方法，一般是系统定义名字 ，类似 \_\_init\_\_()之类的。
  - _foo: 以单下划线开头的表示的是 protected 类型的变量，即保护类型只能允许其本身与子类进行访问，不能用于 from module import \*。
  - __foo: 双下划线的表示的是私有类型(private)的变量, 只能是允许这个类本身进行访问了。

## 关于PyOrthanc

PyOrthanc包装了Orthanc的REST API来更方便实用的使用数据。

### 访问患者、研究、系列和实例信息

```python
from pyorthanc import Orthanc

orthanc = Orthanc('http://localhost:8042')
orthanc.setup_credentials('username', 'password')  # If needed

# To get patients identifier and main information
patients_identifiers = orthanc.get_patients()

for patient_identifier in patients_identifiers:
    patient_information = orthanc.get_patient_information(patient_identifier)

    patient_name = patient_information['MainDicomTags']['name']
    ...
    study_identifiers = patient_information['Studies']    

# To get patient's studies identifier and main information
for study_identifier in study_identifiers:
    study_information = orthanc.get_study_information(study_identifier)
    
    study_date = study_information['MainDicomTags']['StudyDate']
    ...
    series_identifiers = study_information['Series']

# To get study's series identifier and main information
for series_identifier in series_identifiers:
    series_information = orthanc.get_series_information(series_identifier)
    
    modality = series_information['MainDicomTags']['Modality']
    ...
    instance_identifiers = series_information['Instances']

# and so on ...
for instance_identifier in instance_identifiers:
    instance_information = orthanc.get_instance_information(instance_identifier)
    ...
```

### 患者树

在Orthanc实例中为每一个患者建立一个患者树，多个患者树形成了一个患者树森林，树的每一层为$Patient\rightarrow Study\rightarrow Series\rightarrow Instance$。

```python
from pyorthanc import Orthanc, build_patient_forest

patient_forest = build_patient_forest(
    Orthanc('http://localhost:8042/')
)    

for patient in patient_forest:
    patient_info = patient.get_main_information()
    patient.get_name()
    patient.get_zip()
    ...
    
    for study in patient.get_studies():
        study.get_date()
        study.get_referring_physician_name()
        ...

        for series in study.get_series():
            ...
```

### 向Orthanc服务器上传DICOM文件

```python
from pyorthanc import Orthanc

orthanc = Orthanc('http://localhost:8042')
orthanc.setup_credentials('username', 'password')  # If needed

with open('A_DICOM_INSTANCE_PATH.dcm', 'rb') as file_handler:
    orthanc.post_instances(file_handler.read())
```

### 获取已连接的远程登录状态的列表

```python
from pyorthanc import Orthanc

orthanc = Orthanc('http://localhost:8042')
orthanc.setup_credentials('username', 'password')  # If needed

orthanc.get_modalities()
```

### 从远程登录设备查询（C-Find）和检索（C-Move）

```python
from pyorthanc import RemoteModality, Orthanc

orthanc = Orthanc('http://localhost:8042')
orthanc.setup_credentials('username', 'password')  # If needed

remote_modality = RemoteModality(orthanc, 'modality')

# Query (C-Find) on modality
data = {'Level': 'Study', 'Query': {'PatientID': '*'}}
query_response = remote_modality.query(data=data)

# Retrieve (C-Move) results of query on a target modality (AET)
remote_modality.move(query_response['QUERY_ID'], 'target_modality')
```

### 匿名化患者并获取文件

```python
from pyorthanc import Orthanc

orthanc = Orthanc('http://localhost:8042')
orthanc.setup_credentials('username', 'password')  # If needed

a_patient_identifier = orthanc.get_patients()[0]

orthanc.anonymize_patient(a_patient_identifier)

# result is: (you can retrieve DICOM file from ID)
# {'ID': 'dd41f2f1-24838e1e-f01746fc-9715072f-189eb0a2',
#  'Path': '/patients/dd41f2f1-24838e1e-f01746fc-9715072f-189eb0a2',
#  'PatientID': 'dd41f2f1-24838e1e-f01746fc-9715072f-189eb0a2',
#  'Type': 'Patient'}
```

# 关于整体思路

## Orthanc的接口

官方文档中的演示均通过curl完成。

### 内容访问

- 列出不同层次的Orthanc资源，Orthanc中包含四种资源类型：Patient、Study、Series、Instance

  ```
  $ curl http://localhost:8042/patients
  $ curl http://localhost:8042/studies
  $ curl http://localhost:8042/series
  $ curl http://localhost:8042/instances
  ```

- 通过ID访问单个的某资源，可通过患者信息向下逐步浏览直至到实例层

  ```
  $ curl localhost:8042/patients/8c4d0f60-4c7b0273-785ba198-aded6ba3-a47a89a7
  $ curl http://localhost:8042/studies/9ad2b0da-a406c43c-6e0df76d-1204b86f-78d12c15
  $ curl localhost:8042/series/9b9a96f5-f8d4a65c-9666d5b0-cc5eba6b-d84abd95
  $ curl localhost:8042/instances/5dbdad22-d21b8e67-83358204-fc591c97-55571dda
  ```

### 返回JSON文件

不同的资源类型返回的JSON文件不同，但都包含ID、MainDicomTags和Type。可以将JSON文件转化为字典，便于操作。

### 上传和下载DICOM文件

- 上传

```
$ curl -X POST http://localhost:8042/instances --data-binary @CT.X.1.2.276.0.7230010.dcm
```

- 下载

```
$ curl http://localhost:8042/instances/5dbdad22-d21b8e67-83358204-fc591c97-55571dda/file > Instance.dcm
```

## Requests库的使用方法

可能使用的方法主要有get()、post()、put()和delete()，以及一些HTTP的错误和异常处理。

## python库模块化开发的方式

- 建立package
- 具体分为哪几个类
- 编写每个类的属性和接口，并编写接口文档
- 实现模块，使用私有函数实现不被外部调用的模块函数
- 更改\_\_init\_\_.py
- 编写测试代码，提供测试文件

## 需求分析

- 实现get()、put()、post()和delete()方法
- 实现patient、study、series和instance的上述四种方法

## 类、属性、方法



## 接口文档

