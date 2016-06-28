OK OpenAPI Gateway Manual
===

# 术语表

### app

调API的应用，可以是手机端App，也可以是一个服务器端程序

### appkey

App的唯一身份标识

### api id

API的唯一身份标识，名字相同的API，不同版本号对应不同的API ID

### 开发者

开发App的人

### 后端服务

在网关后面，提供实际业务功能的服务，如B2C网站的商品查看、创建订单。

服务可以是一个本地的PHP类方法，也可以是一个远程的RPC接口

### secret

客户端与服务端通信的密钥，双方使用相同的密钥对请求参数做消息摘要，如果摘要结果相同，说明请求参数传输过程未遭篡改。

注意，它不是任何用户的密码，不能用于登录。

# 0 亮点
## 成功案例

### 可溯金融
- https://dai.kesucorp.com
- 农业金融领域领跑者，年成交额10+亿
- 2016年5月和蚂蚁金服、京东金融一起接受财经国家周刊专访。《财经国家周刊》是新华社推出的第一本财经类期刊。
- 使用本产品，两个月实现PC版代码的API化和iOS/Android APP上线，随后业绩增长10倍。至今稳定运行一年多。

### OpsKitchen
- https://ops.best, https://www.OpsKitchen.com
- 全球领先的自动化运维平台
- 全站所有功能都基于OpenAPI，稳定运行一年多，有300个开放的API

## 设计精巧，架构先进
- 架构与Taobao, Tencent的开放平台接近，虽是为中小企业而生，但若为其配备Web自助服务界面，可用作大型互联网公司的开放平台。
- 作者系淘宝开放平台的元老级负责人，精通开放平台产品设计，同时知晓当前互联网巨头开放平台的痛点。
- 没有历史包袱，实践了一些先进的架构与功能。

## 可配置、定制性强
- 暴露给客户端的参数名都可以配置，体现个性化，提高安全性
- 参与签名的参数数量、顺序可以配置
- 系统参数的数据格式可以配置
- AJAX跨域可以启用和禁用
- 数据来源（GET，POST还是COOKIE）可配置
- 可以自行开发Hessian, Raw HTTP, XML-RPC等通信adapter与现有系统对接，只要写一个类即可

## 后端服务接入门槛低
- 多语言支持，凡是支持Thrift的语言都支持
- 也可以直接写本地PHP，不用RPC构架，特别适合时间紧、开发人员少的初创产品
- 一切配置定义都在数据库里，有美观易操作的API管理界面，无须手工书写任何配置文件
- 后端服务提供者写代码时，甚至都感觉不到是在对外提供API，就像写一个普通的类一样，只要返回值是标准格式有errorCode, data等字段即可

## 国际化
- 并非专门针对某一国家地区设计，对客户所在地区没有任何假设和限制
- 使用unix timestamp，不包含时区信息，客户可同时为多个不同时区的调用者服务，而不会产生时间换算的问题

## 安全、健壮
- 防URL重放攻击
- 防SQL注射攻击
- 签名有时效性
- 自动检验输入参数，不信任任何客户端输入
- 灵活、细粒度的访问权限控制，只有经过授权才能访问
- 灵活、细粒度的限问频率限制，保护后端服务提供者
- 无论后端发生什么异常（PHP语法错误、数据库繁忙，未知异常），都不影响网关返回错误码和错误消息给客户端和记录日志


## 高性能
一台1CPU、1G内存的虚拟机，每天可承担230万次访问，峰值可达160 QPS

## 保持稳定，不断创新
自问世以来，每个星期都有代码提交，每月都有创新功能，但对客户来说，网关的设计是稳定的，版本升级时，后端服务代码不用做任何变更，网关维护人员也仅仅需要合并代码和对网关数据库做一个数据迁移。

有一些专利功能是BAT之类的巨头开放平台也不具备的，比如：

### 无缝更换secret
更新secret时，允许旧secret继续存在，在指定时间段内，使用旧secret和新secret的用户都可以正常访问，到达指定的时刻，旧secret才会失效。

这样，如果客户觉得自己某个html 5产品的secret可能已经非法窃取，无须等到凌晨业务量小的时候更新secret，任何时候都可以更新secret。

### 浏览器端不包含明文Secret
采用专利技术，兼容主流的md5摘要算法，但又不在javascript中包含明文的secret，任何破解方法都不可能通过javascript客户端代码逆向运算破解出secret。

结合上述无缝更换secret功能，和签名有效期功能，可将平台appkey, secret泄漏的安全风险降到最低。

### 后端入参无限多对象嵌套
假设User, Addreee, Contact都是Class，Address和Contact又是User的属性，后端代码可以定义这样的入参：

	public createOrder(User buyer, User seller...)
	


## 源代码质量高，可读性强
- 通过IntelliJ IDEA的代码质量检测，无任何错误
- 代码风格好，比如几乎没有magic number和magic string


# 1 功能

以下所有功能，都不支持开发者自助操作。如：实时申请app，更新密钥，申请Api权限。而是由开发者线下提申请（比如通过邮件，甚至是签署合同），平台运营方去操作数据库实现变更。

这是因为，只有真正的大企业开放平台（如淘宝、支付宝、微信），才会有大量频繁的开发者请求，才会做一个自助服务台给开发者使用。中小企业的开发者和应用数量有限，变更频率也低，开发者自助服务台不是刚需。

## 1.1 App

- 可以支持无限多个app
- app可以随时被禁用、解禁
- app的密钥可以更新
- app信息记载在数据库中，新增app、禁用/解禁、更新密钥都通过操作数据库实现

## 1.2 Api

### 1.2.1 多版本

一个API可以有多个版本，如user.login 1.0，user.login 2.0，

名字加版本共同作为API的唯一身份标识，名字相同版本不同的API，是两个完全独立的API，其后的参数配置、权限、频率控制都是完全独立的
### 1.2.2 API与后端服务灵活映射

- 每个API都可以定义自己的后端服务类型、class名（带namesapce的完整class name）、method名
- 不同API可以对应同一个后端服务
- API名字与后端服务的名字没有任何潜规则限制，映射关系存储在数据库中
- API名字一旦确定不能随便改，后端服务可以随意重构

### 1.2.3 输入参数

支持三类输入参数：

#### 1.2.3.1 HTTP HEADR里的系统参数

系统参数与API不相关，每个API都有这些系统参数，参数名由网关定义，后端服务提供者只能获取，不能定义。系统参数的验证规则（Validator Class）也由网关定义。

包含两种：

1. 标准的HTTP HEADER，如User-Agent
2. 平台自定义的HTTP HEADER，如OA-Sign

技术上讲，平台自定义的HTTP HEADER参数，放到HTTP BODY里去，也是可行的，为什么要放到HEADER里呢？两个原因：

- 第一，是为了避免URL重放攻击，提升安全性
- 第二，规范客户端的开发，HTTP HEADER里的参数，基本都是写死在app的配置里的，每次api请求都一样，如appkey, app version。只有两个例外：signature和session id，它是变化的，但为了安全，放到HEADER里了



以下是完整的HEADER系统参数列表：

    const APP_KEY        = "OA-App-Key"; #appkey
    const APP_VERSION    = "OA-App-Version"; #客户端版本
    const DEVICE_ID      = "OA-Device-Id"; #设备ID
    const MARKET_ID      = "OA-App-Market-ID"; #客户端渠道ID
    const SESSION_ID     = "OA-Session-Id"; #Session ID
    const SIGNATURE      = "OA-Sign"; #签名



#### 1.2.3.2 HTTP BODY里的系统参数

系统参数与API不相关，每个API都有这些系统参数，参数名由网关定义，后端服务提供者只能获取，不能定义。系统参数的验证规则（Validator Class）也由网关定义。

调用不同API时，这些系统参数的名字一样，值不一样，它们包括：

###### api

API的名字，如user.login

###### version

API的版本，如2.0

###### timestamp

时间戳，发起请求的时间

###### callback

如果返回格式是jsonp，用这个参数指定callback函数名

###### params

是一个json字串，就是API自定义的业务参数

##### 1.2.3.3 HTTP BODY里的业务参数

app通过HTTP BODY提交上来的业务参数，个数不限，数据类型不限，参数名不限，嵌套深度不限。

其具体实现是一个json串化后的kv列表，放在HTTP BODY的params字段中，如: {"username":"qinjx","password":"123456"}

- 每个API的业务参数的参数名是在数据库里定义的
- 每个业务参数的验证规则是在数据库里定义的，可以输出给客户端，保证前后端验证规则统一
## 1.3 后端服务

定义API时，可以定义此API对应的后端服务，这些定义信息都存储在数据库里。

### 1.3.1 后端服务类型

API可定义的后端服务类型包括：

#### 1.3.1.1 本地PHP

即，要运行的后端服务代码就在API网关这台机器上，网关可以直接在内存里调用这些方法，不需要网络通信，不需要数据编码解码

##### 优点

- 性能高
- 简单，对既有业务侵入小，提供后端服务的人不需要学习和使用thrift, grpc等rpc框架，可以满足大多数小企业的需求

##### 缺点

- 后端服务只能是PHP写的（因为网关是PHP写的），通过类似quercus一类的JVM Base PHP runtime也有实现PHP本地调Java，但不是业界主流
- 对大一点的系统来说，构架不优雅，API网关机器要有访问业务系统数据库的权限，还要配置后端服务依赖的环境（例如config, DI service），后端服务代码与网关代码会竞争系统资源（CPU、内存），影响网关稳定性

#### 1.3.1.2 Thrift远程服务

即，后端服务代码运行在另外一台机器上，网关要通过thrfit这个RPC框架去调用它

优缺点刚好跟“本地PHP”相反

#### 1.3.1.3 自定义的RPC协议
若已在生产环境使用了RPC协议，但又不是Thrift，可以加入自定义的Service Protocol Adapter，只要继承ServiceProtocolAbstract抽象类即可

##### 优点

- 后端服务可以用任何thrift支持的语言开发，除了支持PHP，thrift还支持：老牌主流编程语言C/C++, Java, C#；新贵Go, Haskell；脚本语言Python, Nodejs, Ruby, Perl，更多语言参见这个列表：https://github.com/apache/thrift/tree/master/tutorial
- 架构优雅，网关与后端服务部署在不同的服务器，稳定和安全性好

##### 缺点

- 性能略差，差多少取决于rpc框架，socket模式下的thrift会比xmlrpc性能好
- 后端服务开发人员要学习一种rpc框架才能把服务提供给网关

### 1.3.2 后端服务的参数

#### 1.3.2.1 参数的数据类型

可以是简单的标量，如：

- 数字
- 字符串
- 布尔值

也可以是复杂的结构体，如：

- Array
- hashMap
- DomainObject

##### 1.3.2.2 参数的顺序

顺序保存在数据库中，可以随时调整

注：hashMap和DomainObject中属性寻址靠的是string key，不需要顺序

#### 1.3.3 后端服务参数与用户输入参数的映射

每一个参数、结构体的每一个key都可以在数据库中定义映射关系，映射关系的要素包括：

- 数据来源
- 数据类型
- 默认值

##### 1.3.3.1 数据来源

数据来源分4种，除了3种用户输入的参数外，还支持从服务端SESSION里取出数据，映射给后端服务，4种数据来源如下：

1. 通过验证和过滤的HTTP HEADER系统参数，除了HTTP客户端上传的HEADER外，也可以获取Web Server预定义的标准HEADER，如REMOTE_ADDR, REMOTE_PORT
2. 通过验证和过滤的HTTP BODY系统参数
3. 通过验证和过滤的HTTP BODY业务参数
4. 无需验证的SESSION数据

##### 1.3.3.2 数据类型

Java、C++等强类型语言的对数据要求较严格，而用户通过HTTP传过来的原始数据本质上都是字符串，需要网关强制转化一下再传递给后端服务

##### 1.3.3.3 默认值

Java使用方法名和参数类型作为方法的signature，不允许两个方法拥有相同的signature，因此Java里的方法没有默认值，因此如果后端服务是Java开发，不能像PHP一样在定义方法时，把默认参数写在形参里，如果需要默认值，由网关实现。

## 1.4 访问控制

### 1.4.1 权限

采用白名单机制，除非明确授权，任何App都没有权限访问任何API

##### 1.4.1.1 权限要素

###### Appkey

App的唯一身份标识

###### Api Id

user.login 1.0和user.login 2.0是两个不同的api id，他们的权限规则是彼此独立的。


### 1.4.2 匿名访问

定义API时，可以指定此API是否允许匿名访问，若不允许，则网关会检查请求的HTTP HEADER中是否有session id，以及此session是否有效

### 1.4.3 调用频率

采用黑名单机制，若未明确设置，则表示此API的调用频率是无限的

#### 1.4.3.1 规则要素

频率限制由以下要素构成

##### Appkey

App的唯一身份标识

##### Api Id

user.login 1.0和user.login 2.0是两个不同的api id，他们的权限规则是彼此独立的。

##### 时间窗口

时间窗口定义了一个时间长度（不是具体的时刻，只是一个时间长度，如5分钟，1小时），在此时间段内，累计调用次数不能超过平台设定的限制，不同时间窗口间的调用次数不会累计。假设时间窗口是5分钟，则累计调用次数每5分钟清零一次。最小时间窗口是1秒

##### 最大可调用次数

在规定的时间窗口内，可以调用的最大次数。

注意，是总次数，不是每秒多少次，也不是每分钟多少次。

### 1.4.4 签名

#### 1.4.4.1 签名正确性

网关会自动校验客户端发送的签名是否正确，如校验不通过，直接返回错误，后续的所有业务流程都不会触发。

#### 1.4.4.2 签名有效期

每个API可以定义自己的签名有效期，精确到秒。若签名内容正确，但超过了有效期，网关也会直接返回错误。这是为了保护一些敏感API（如创建订单、修改密码），防止客户端签名泄露引发安全风险。

## 1.5 网关工作流程

网关依次执行如下操作，遇到错误即返回错误码给客户端，并停止执行后面的处理流程

- 校验HTTP HEADER中的系统参数

- 校验HTTP BODY中的系统参数

- 从数据库中查询app的信息，主要是secret, 是否禁用两个信息

- 检查签名有效性

- 从数据库中获取API信息，若API存在，检查是否标识为【已废弃】

- 检查签名是否过期（客户端传递的timestamp不能大于当前时间，否则会在【校验HTTP BODY中的系统参数】这一步失败）

- 检查用户SESSION

- 若API配置了必须登录才能访问，而用户又没登录，则返回错误。

- 检查是否有权限调此API

- 检查调用次数是否超限

- 检查访问频率是否超过后端系统的承受上限

- 校验HTTP BODY中的业务参数

- 从数据库中获取后端服务定义、拼装参数

- 调用后端服务

- 判断后端服务返回的结果

- 输出结果给客户端

- 捕获后端服务的错误、异常（若后端服务以本地PHP代码的形式提供，后端服务中很可能会存在Fatal Error，甚至是一些无意中写下的die, exit代码，这些都会导致网关的后续代码得不到执行。因此需要在php script执行完成时捕获这些异常和错误，无论是成功完成，还是出错退出，客户端一定会收到结果，日志一定会得到记录）

- 更新API实时调用频率

- 写API访问日志

## 1.6 日志

在将结果发送给客户端之后，网关会把本次请求记入日志。

### 1.6.1 成功日志

成功日志包含如下字段：

- appkey
- api id
- 访问时间
- 客户端ip
- 客户端版本
- 客户端渠道ID

### 1.6.2 错误日志

错误日志包含如下字段：

- appkey
- api id
- 访问时间
- 客户端ip
- 网关错误码
- 后端服务返回的子错误码

### 1.6.3 错误详情
如果定义API时设置了这个API需要保存错误详情，则在出错时会把客户端请求的数据（含 header和body）、服务端返回的数据都保存到一个单独的表中，便于后端服务开发者诊断错误。

这个功能特别适用于调用量少但极其重要的API，比如注册用户、创建订单。

## 1.7 其它

### 1.7.1 自定义配置
平台运营方可以自定义

#### 1.7.1.1 输出格式支持

支持json和jsonp两种，分别对应不同的网关url，如：

api.ok.com/gw/json
api.ok.com/gw/jsonp

#### 1.7.1.2 系统参数名
HTTP HEADER和HTTP BODY里的参数名都可以自定义

#### 1.7.1.3 参与签名的参数和顺序


# 3 性能
## 3.1 性能测试环境
一个VirtualBox虚拟机，1个虚拟处理器，1G内存

宿主机是一台MacBook Pro, 2.7G Intel Core i5（双核），8G内存，128G SSD

## 3.2 测试用的后端API
用于测试的API只返回当前时间（Unix Timestamp），无其它任何操作，无硬盘IO，CPU消耗极小

## 3.3 性能调优设置
- 开启PHP Opcache
- 开启网关的Cache，把api, app等数据库里读取的信息缓存到本地硬盘上
- 不缓存任何后端服务的业务数据，即每次请求都会真正地转发到后端服务上

## 3.4 测试数据
使用ab测试，12并发，总计2万个请求，160 QPS

假设每天用户访问集中在上午10点到晚上10点这12小时，峰值并发数是平均值的3倍，则这种配置的网关服务器，一天可以承担160 * 46200 / 3 = 230万次API请求

# 4 规划中子系统
## 4.1 Admin Console
平台管理者控制台，用于：

### 4.1.1 管理software和app

#### 4.1.1.1 管理app
- 禁用app
- 更新secret

#### 4.1.1.2 管理software（已完成）
- 授予访问api的权限
- 设置api访问频率

### 4.1.2 管理api（已完成）
- 新增、编辑、下线api
- 设置api参数
- 映射后端服务

## 4.2 App Developer Tools
### 4.2.1 Doc
查看自动生成的api文档

### 4.2.2 Mock System
依照与服务端开发者的约定，根据不同请求串获得不同的mock数据

### 4.2.3 Online Test Tool
浏览器端的api demo，只在网页表单中填入业务参数，即可查看api调用效果

### 4.2.4 SDK Model Auto generator
自动生成的SDK Model（主要是Request和Response对象），能让客户端代码更优雅、健壮。

## 4.3 Dashboard
查看API报表、调用量走势、成功率、错误率、错误分布

## 4.4 Access Token
类似银行的叫号机，实现API的幂等性

## 4.5 Captcha
图片验证码，考虑异步预先生成图片，而不是客户端请求才生成

# 5 代码示例
## 5.1 API定义
存储在数据库中，这里显示的是数据库取出来然后JSON编码的结果

	{
		id: "1008",
		catId: "3",
		name: "ops.meta.repo.add",
		version: "1.0",
		deprecated: "0",
		signatureTtl: "60",
		maxQps: "0",
		saveErrorDetail: "1",
		sharedSpoId: "6",
		serviceProtocol: "local_php",
		serviceProtocolOption: {
			className: "Ops\Lib\Service\Meta\RepoService",
			methodName: "createRepo"
		},
		sessionOption: "CHECK_USER_LOGIN"
	}
## 5.2 客户端调用

### 5.2.1 JS示例

    angular.module 'app.controllers'

    .controller 'OpsMetaRepoAddCtrl', ($scope, $location, ApiService) ->
      $scope.saveRepo = (repo) ->
        ApiService.callApi
          api:
            name: "ops.meta.repo.add"
            version: "1.0"
          params: $scope.repo
          successCallBack: (data) ->
            $location.path("/ops_meta/repo_detail").search("id", data)

### 5.2.2 Go示例

	param := make(map[string]string)
	param["osReleaseId"] = "3022"
	resp, err = client.CallApi("ops.meta.osImage.listByOsReleaseId", "1.0", param)
	if err != nil {
		fmt.Println(err.Error())
	} else {
		fmt.Println(resp)
	}
	
完整Go示例源码：https://github.com/OpsKitchen/ok_api_sdk_go/blob/master/example.go

## 5.3 服务端

    /**
     * @param Repo $repo
     * @return ServiceResultDO data: int
     */
    public function createRepo(Repo $repo)
    {
        if (Repo::findUniqueByUK(["name" => $repo->getName()])) {
            return new FailureResultDO(new MetaErrorEnum(MetaErrorEnum::REPO_NAME_ALREADY_EXISTS));
        }

        if ($repo->create()) {
            return new SuccessResultDO($repo->getId());
        } else {
            return new FailureResultDO(new CommonErrorEnum(CommonErrorEnum::INTERNAL_SERVER_ERROR));
        }
    }

# 6 SDK
## 6.1 Golang
https://github.com/OpsKitchen/ok_api_sdk_go
