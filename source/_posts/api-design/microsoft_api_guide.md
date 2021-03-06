---


title: Microsoft REST API 准则
author: kevin
date: 2020-09-01 16:20:00
updated: 2020-09-01 16:20:00
tags:
- api-design
---

## Microsoft REST API准则工作组

| 名称                         | 名称                            | 名称                                  |
| ---------------------------- | ------------------------------- | ------------------------------------- |
| 戴夫·坎贝尔（CTO C + E）     | 里克·拉希德（CTO ASG）          | John Shewchuk（TED总部技术研究员）    |
| 马克·鲁西诺维奇（CTO Azure） | Steve Lucco（DevDiv技术研究员） | Murali Krishnaprasad（Azure应用平台） |
| 罗伯·霍华德（ASG）           | 彼得·托（OSG）                  | 克里斯·穆林斯（ASG）                  |

文档编辑：John Gossman（C + E），Chris Mullins（ASG），Gareth Jones（ASG），Rob Dolin（C + E），Mark Stafford（C + E）

## 1. 摘要

作为设计原则，Microsoft REST API准则鼓励应用程序开发人员通过RESTful HTTP接口访问资源。为了在遵循Microsoft REST API准则的平台上为开发人员提供尽可能流畅的体验，
REST API应当遵循一致的设计准则，以使其使用起来简单直观。

本文档建立了Microsoft REST API应该遵循的准则，以便一致地开发RESTful接口。

<!-- more -->

## 3. 引言

开发人员通过HTTP接口访问大多数Microsoft Cloud Platform资源。尽管每个服务通常都提供特定于语言的框架来包装其API，但其所有操作最终都归结为HTTP请求。
Microsoft必须支持广泛的客户端和服务，并且不能依赖可用于每个开发环境的丰富框架。因此，这些准则的目标是确保具有基本HTTP支持的任何客户端都可以轻松，一致地使用Microsoft REST API。

为了为开发人员提供尽可能流畅的体验，使这些API遵循一致的设计准则非常重要，因此使它们使用起来简单直观。本文档建立了Microsoft REST API开发人员应遵循的准则，以一致地开发此类API。

总的来说，一致性的好处也会产生；一致性使团队可以利用通用代码，模式，文档和设计决策。

这些准则旨在实现以下目标：

- 为Microsoft的所有API端点定义一致的做法和模式。
- 尽可能严格遵守业界公认的REST / HTTP最佳实践。*
- 使所有应用程序开发人员都可以通过REST界面轻松访问Microsoft Services。
- 允许服务开发人员利用其他服务的先前工作来实施，测试和记录一致定义的REST端点。
- 允许合作伙伴（例如，非Microsoft实体）将这些准则用于他们自己的REST端点设计。

**注意**：准则旨在与符合REST体系结构样式的建筑服务保持一致，尽管它们并未解决或要求遵循REST约束的建筑服务。在整个文档中，术语“ REST”用于表示符合REST精神的服务，而不是本书所遵循的REST

### 3.1 推荐阅读

为了开发良好的基于HTTP的服务，建议理解REST体系结构风格的原理。如果您不熟悉RESTful设计，这里有一些不错的资源：

[Wikipedia上的REST](http://en.wikipedia.org/wiki/Representational_state_transfer) -REST背后的常见定义和核心思想概述。

[REST论文](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm) -Roy Fielding论文中有关REST的一章，即“建筑风格和基于网络的软件体系结构的设计”

[RFC 7231-](https://tools.ietf.org/html/rfc7231)定义HTTP / 1.1语义的规范，并被视为权威资源。

[实践中](http://www.amazon.com/REST-Practice-Hypermedia-Systems-Architecture/dp/0596805829/)的REST-关于REST基础的书籍。

## 4 解释准则

### 4.1 准则的适用

这些准则适用于Microsoft或任何合作伙伴服务公开公开的任何REST API。私有或内部API也应尝试遵循这些准则，因为内部服务最终倾向于公开公开。
一致性不仅对外部客户而且对内部服务消费者都很有价值，并且这些准则提供了对任何服务都有用的最佳实践。

有正当理由免除这些准则。显然，实现或必须与某些外部定义的REST API互操作的REST服务必须与该API兼容，而不一定与这些准则兼容。
一些服务可能还具有特殊的性能需求，这些需求需要不同的格式，例如二进制协议。

### 4.2 现有服务和服务版本指导

我们不建议仅出于合规性考虑而对早于这些准则的服务进行重大更改。当兼容性遭到破坏时，该服务应尝试在下一版本中变得兼容。
当服务添加新的API时，该API应该与相同版本的其他API一致。因此，如果针对该指南的1.0版编写了服务，则应将增量添加到该服务的新API也遵循1.0版。
然后可以在服务的下一个主要版本中升级该服务以使其与指南的最新版本保持一致。

### 4.3 需求语言

本文档中的关键字“必须”，“不得”，“必须”，“应”，“应不”，“应”，“不应”，“推荐”，“可以”和“可选”按照[RFC 2119中的](https://www.ietf.org/rfc/rfc2119.txt)描述进行解释。

### 4.4 许可证

该作品已根据知识共享署名4.0国际许可协议获得许可。要查看此许可证的副本，请访问 [http://creativecommons.org/licenses/by/4.0/])
 或向美国纽约州94042，山景城的PO Box 1866邮政局（Creative Commons）发信。

## 5 分类

作为加入Microsoft REST API指南的一部分，服务必须符合以下定义的分类法。

### 5.1 错误

错误，或更具体地说是服务错误，定义为客户端将无效数据传递给服务，并且服务**正确**拒绝该数据。
示例包括无效的凭据，不正确的参数，未知的版本ID或类似内容。这些通常是“ 4xx” HTTP错误代码，是客户端传递不正确或无效数据的结果。

错误*不会*影响总体API可用性。

### 5.2 故障

错误，或更具体地说是服务错误，定义为服务无法正确响应有效的客户端请求而返回。这些通常是“ 5xx” HTTP错误代码。

故障*确实*有助于总体API可用性。

由于速率限制或配额失败而失败的呼叫不得视为错误。由于服务快速失败请求而导致失败的调用（通常是出于自身保护的考虑）确实被视为错误。

### 5.3 延迟

延迟定义为特定API调用完成所需的时间，并尽可能接近客户端。此指标以相同的方式适用于同步和异步API。对于长时间运行的呼叫，
延迟是根据初始请求进行衡量的，并衡量该呼叫（而不是整个操作）需要花费多长时间。

### 5.4完成时间

公开长时间操作的服务必须跟踪这些操作的“完成时间”指标。

### 5.5 长时间运行的API错误

对于长时间运行的API，初始请求可以开始操作，并且检索结果的请求可以在技术上正常工作（每个返回200），但是基础操作失败。
长期运行的故障必须作为故障汇总到总体可用性指标中。

## 6 客户指导

为了确保与REST服务对话的客户获得最佳体验，客户应遵循以下最佳实践：

### 6.1忽略规则

对于松散耦合的客户端，在调用之前尚不清楚数据的确切形状，如果服务器返回了客户端未期望的内容，则客户端必须安全地忽略它。

一些服务可以在不改变版本号的情况下在响应中添加字段。这样做的服务必须在其文档中明确指出，并且客户端必须忽略未知字段。

### 6.2可变顺序规则

客户端不得依赖于数据在JSON服务响应中出现的顺序。例如，客户端应该对JSON对象中字段的重新排序具有弹性。当服务支持时，客户端可以请求以特定顺序返回数据。例如，服务可以支持使用*$ orderBy* querystring参数来指定JSON数组中元素的顺序。服务还可以明确规定某些元素的顺序，作为服务合同的一部分。例如，服务可以始终返回JSON对象的“类型”信息作为对象中的第一个字段，以简化客户端上的响应解析。客户可以依靠服务明确标识的订购行为。

### 6.3静默失败规则

请求可选服务器功能（例如可选标头）的客户端必须对服务器具有弹性，而忽略该特定功能。

## 7 一致性基础

### 7.1 URL结构

人类应该能够轻松地阅读和构造URL。

在没有良好支持的客户端库的情况下，这有助于发现并简化在平台上的采用。

结构良好的URL的示例是：

```text
https://api.contoso.com/v1.0/people/jdoe@contoso.com/inbox
```

不友好的URL是：

```text
https://api.contoso.com/EWS/OData/Users('jdoe@microsoft.com')/Folders('AAMkADdiYzI1MjUzLTk4MjQtNDQ1Yy05YjJkLWNlMzMzYmIzNTY0MwAuAAAAAACzMsPHYH6HQoSwfdpDx-2bAQCXhUk6PC1dS7AERFluCgBfAAABo58UAAA=')
```

经常出现的模式是使用URL作为值。服务可以使用URL作为值。例如，以下是可接受的：

```text
https://api.contoso.com/v1.0/items?url=https://resources.contoso.com/shoes/fancy
```

### 7.2 URL长度

在RFC 7230的[3.1.1](https://tools.ietf.org/html/rfc7230#section-3.1.1)节中定义的HTTP 1.1消息格式在请求行上没有长度限制，该长度包括目标URL。从RFC：

> HTTP并未对请求行的长度设置预定义的限制。[...]接收到比任何希望解析的URI更长的请求目标的服务器，必须以414（URI太长）状态码作为响应。

可以生成超过2083个字符的URL的服务必须为他们希望支持的客户提供便利。以下是一些确定目标客户支持的来源：

- `http://stackoverflow.com/a/417184`
- `https://blogs.msdn.microsoft.com/ieinternals/2014/08/13/url-length-limits/`

另请注意，某些技术堆栈具有严格的可调整的URL限制，因此在设计服务时请记住这一点。

### 7.3规范标识符

除了友好的URL，可以移动或重命名的资源也应该公开包含唯一稳定标识符的URL。与某些服务使用“ /我”快捷方式的情况一样，可能有必要与服务进行交互以从资源的友好名称中获取稳定的URL。

稳定标识符不需要是GUID。

包含规范标识符的URL的示例是：

```text
https://api.contoso.com/v1.0/people/7011042402/inbox
```

### 7.4支持的方法

操作必须尽可能使用正确的HTTP方法，并且必须尊重操作幂等性。HTTP方法通常称为HTTP动词。这些术语在此上下文中是同义词，但是HTTP规范使用术语“方法”。

以下是Microsoft REST服务应支持的方法的列表。并非所有资源都支持所有方法，但是使用以下方法的所有资源必须符合其用法。

| 方法      | 描述                                                         | 是否幂等 |
| --------- | ------------------------------------------------------------ | -------- |
| `GET`     | 返回对象的当前值                                             | 是       |
| `PUT`     | 替换对象或创建命名对象（如果适用）                           | 是       |
| `DELETE`  | 删除物件                                                     | 是       |
| `POST`    | 根据提供的数据创建新对象，或提交命令                         | 否       |
| `HEAD`    | 返回对象的元数据以获取GET响应。支持GET方法的资源也可以支持HEAD方法 | 是       |
| `PATCH`   | 对对象应用部分更新                                           | 否       |
| `OPTIONS` | 获取有关请求的信息；有关详情，请参见下文。                   | 是       |

#### 7.4.1 POST

POST操作应支持Location响应标头，以通过Location标头指定未显式命名的任何已创建资源的位置。

例如，设想一个允许创建托管服务器的服务，该服务将由该服务命名：

```text
POST http://api.contoso.com/account1/servers
```

响应：

```text
201 Created
Location: http://api.contoso.com/account1/servers/server321
```

其中 “server321”是服务分配的服务器名称。

服务还可以在响应中返回所创建项目的完整元数据。

#### 7.4.2 PATCH

IETF已将PATCH标准化为用于增量更新现有对象的方法（请参阅[RFC 5789](http://tools.ietf.org/html/rfc5789)）。符合Microsoft REST API准则的API应该支持PATCH。

#### 7.4.3通过PATCH创建资源（UPSERT语义）

允许调用者在create上指定键值的服务应支持UPSERT语义，而那些必须支持使用PATCH创建资源的服务。由于PUT被定义为内容的完全替代，因此使用PUT修改数据对客户端来说很危险。不了解（并因此忽略）资源属性的客户端在尝试更新资源时不太可能在PUT上提供它们，因此可能会无意中删除这些属性。服务可以选择支持PUT更新现有资源，但如果这样做，则它们必须使用替换语义（即，在PUT之后，资源的属性必须与请求中提供的属性匹配，包括删除未提供的任何服务器属性）。

在UPSERT语义下，服务器将对不存在资源的PATCH调用作为“创建”处理，而对现有资源的PATCH调用则作为“更新”处理。为了确保更新请求不被视为创建请求，反之亦然，客户端可以在请求中指定前置HTTP头。如果服务包含If-Match标头，则服务不得将PATCH请求视为插入；如果服务包含包含值为“ *”的If-None-Match标头，则服务不得将PATCH请求视为更新。

如果服务不支持UPSERT，则对不存在的资源的PATCH调用务必导致HTTP“ 409冲突”错误。

#### 7.4.4 Options 和链接头

OPTIONS允许客户端至少通过返回表示该资源有效方法的Allow标头来检索有关资源的信息。

另外，服务应该包括一个链接头（参见[RFC 5988](http://tools.ietf.org/html/rfc5988)），以指向相关资源的文档：

```text
Link: <{help}>; rel="help"
```

其中{help}是文档资源的URL。

有关使用OPTIONS的示例，请参阅[预检CORS跨域调用](http://www.w3.org/TR/cors/#resource-preflight-requests)。

### 7.5标准请求头

Microsoft REST API指南服务应使用下面的请求标头表。没有强制使用这些头，但是如果使用，则必须始终使用它们。

所有报头值必须遵循规范中规定的语法规则，其中定义了报头字段。[RFC7231](https://tools.ietf.org/html/rfc7231)中定义了许多HTTP标头，但是可以在[IANA标头注册表中](http://www.iana.org/assignments/message-headers/message-headers.xhtml)找到批准标头的完整列表。”
<!-- markdownlint-disable MD013 MD033-->
| Header                                   | 类型                                  | 描述                                                         |
| ---------------------------------------- | ------------------------------------- | ------------------------------------------------------------ |
| `Authorization`                          | String                                | 请求的授权标头                                               |
| `Date`                                   | Date                                  | 基于客户端时钟的请求时间戳，采用[RFC 5322](https://tools.ietf.org/html/rfc5322#section-3.3)日期和时间格式。服务器不应该对客户端时钟的准确性做任何假设。该报头可以包含在请求中，但提供时必须采用这种格式。提供此标头时，必须将格林威治标准时间（GMT）用作该标头的时区参考。例如：`Wed, 24 Aug 2016 18:41:30 GMT`。请注意，为此目的，GMT完全等于UTC（世界标准时间）。 |
| `Accept`                                 | 内容类型                              | 请求的响应内容类型，例如：应用程序/ xml文字/ xml应用程序/ json文字/ JavaScript（用于JSONP）根据HTTP准则，这只是一个提示，响应可能具有不同的内容类型，例如blob提取，其中成功的响应将只是blob流作为有效负载。对于遵循OData的服务，应遵循OData中指定的优先顺序。 |
| `Accept-Encoding`                        | Gzip, deflate                         | 如果适用，REST端点应该支持GZIP和DEFLATE编码。对于非常大的资源，服务可以忽略并返回未压缩的数据。 |
| `Accept-Language`                        | "en", "es", etc.                      | 指定响应的首选语言。不需要服务来支持此功能，但是如果服务支持本地化，则必须通过Accept-Language标头来实现。 |
| `Accept-Charset`                         | 字符集类型，例如“ UTF-8”              | 默认值为UTF-8，但是服务应该能够处理ISO-8859-1。              |
| `Content-Type`                           | 内容类型                              | 请求主体的MIME类型（PUT / POST / PATCH）                     |
| `Prefer`                                 | return=minimal，return=representation | 如果指定了return = minimal首选项，则服务应响应成功的插入或更新而返回空主体。如果指定了return = representation，则服务应在响应中返回创建或更新的资源。如果服务具有客户端有时会从响应中受益的场景，但有时响应会对带宽造成太大的影响，则服务应支持此标头。 |
| `If-Match`, `If-None-Match` , `If-Range` | String                                | 支持使用乐观并发控制更新资源的服务必须支持If-Match标头。服务也可以使用其他与ETag相关的标头，只要它们遵循HTTP规范即可。 |
<!-- markdownlint-restore -->
### 7.6标准响应头

服务应返回以下响应标头，除非在“必需”列中注明。
<!-- markdownlint-disable MD013 MD033-->
| 响应Header           | 必要条件                   | 说明                                                         |
| -------------------- | -------------------------- | ------------------------------------------------------------ |
| `Date`               | 所有响应                   | 根据服务器的时钟，以[RFC 5322](https://tools.ietf.org/html/rfc5322#section-3.3)日期和时间格式处理响应的时间戳。该头必须包含在响应中。格林威治标准时间（GMT）必须用作此标头的时区参考。例如：`Wed, 24 Aug 2016 18:41:30 GMT`。请注意，为此目的，GMT完全等于UTC（世界标准时间）。 |
| `Content-Type`       | 所有响应                   | 内容类型                                                     |
| `Content-Encoding`   | 所有响应                   | GZIP或DEFLATE（视情况而定）                                  |
| `Preference-Applied` | 在请求中指定时             | 是否应用了“首选”请求标头中指示的首选项                       |
| `ETag`               | 当请求的资源具有实体标签时 | ETag响应标头字段提供所请求变体的实体标签的当前值。与If-Match，If-None-Match和If-Range一起使用以实现乐观并发控制。 |
<!-- markdownlint-restore -->
### 7.7自定义 Header

给定API的基本操作必须不需要自定义标头。

本文档中的一些准则规定了非标准HTTP标头的使用。另外，某些服务可能需要添加额外的功能，这些功能通过HTTP标头公开。以下准则有助于在使用自定义标头时保持一致性。

不是标准HTTP标头的标头必须具有以下两种格式之一：

1. 在IANA（[RFC 3864](http://www.ietf.org/rfc/rfc3864.txt)）中注册为“临时”的标头的通用格式
2. 标头的范围格式过于针对使用情况而无法注册

下面介绍这两种格式。

### 7.8指定标头作为查询参数

在某些情况下，例如AJAX客户端，某些标头会带来挑战，尤其是在进行跨域调用时，可能不支持添加标头。这样，除了标头之外，还可以将某些标头作为查询参数接受，其命名与标头相同：

并非所有标头都适合作为查询参数，包括大多数标准HTTP标头。

考虑何时接受标头作为参数的标准是：

1. 任何自定义标头也必须被接受为参数。
2. 必需的标准标头可以作为参数接受。
3. 具有安全敏感性的必需标头（例如，授权标头）可能不适合作为参数；服务所有者应根据具体情况进行评估。

此规则的一个例外是Accept标头。通常的做法是使用具有简单名称的方案，而不是HTTP规范中针对Accept所描述的全部功能。

### 7.9 PII参数

与其组织的隐私权政策一致，客户端不应在URL中（路径或查询字符串的一部分）传输URL中的个人身份信息（PII）参数，因为此信息可能会通过客户端，网络和服务器日志以及其他机制被无意间公开。

因此，服务应该接受作为报头发送的PII参数。

但是，由于客户端或软件的限制，在许多情况下无法遵循上述建议。为了解决这些限制，服务还应该接受这些PII参数作为URL的一部分，并与本指南的其余部分保持一致。

接受PII参数的服务（无论是在URL中还是作为标头）均应遵守其组织的工程领导者指定的隐私权政策。这通常包括建议客户端选择标头进行传输，并且实现应遵循特殊的预防措施，以确保正确处理日志和其他服务数据收集。

### 7.10响应格式

为了使组织拥有成功的平台，它们必须以开发人员习惯使用的格式提供数据，并以一致的方式允许开发人员使用通用代码处理响应。

基于Web的通信，尤其是在涉及移动或其他低带宽客户端时，由于多种原因，已朝JSON方向快速发展，包括其重量更轻的趋势以及与基于JavaScript的客户端的易用性。

JSON属性名称应为驼峰式。

服务应提供JSON作为默认编码。

#### 7.10.1客户指定的响应格式

在HTTP中，客户端应使用Accept标头请求响应格式。这只是一个提示，服务器可以选择忽略它，即使这在行为良好的服务器中并不常见。客户端可以发送多个Accept标头，服务可以选择其中之一。

默认的响应格式（不提供Accept标头）应该是application / json，并且所有服务都必须支持application / json。

| 接受的请求头     | 响应类型            | 说明                                     |
| ---------------- | ------------------- | ---------------------------------------- |
| application/json | 返回内容应该是 JSON | 对于 `text/javascript` 可以是 JSONP 格式 |

```text
GET https://api.contoso.com/v1.0/products/user
Accept: application/json
```

#### 7.10.2错误条件响应

对于不成功的情况，开发人员应该能够编写一段代码，以在不同的Microsoft REST API准则服务之间一致地处理错误。
这允许构建简单可靠的基础结构来将异常作为成功响应的独立流程来处理。以下内容基于OData v4 JSON规范。
但是，它非常通用，不需要特定的OData构造。即使没有使用其他OData构造，API也应该使用这种格式。

错误响应必须是单个JSON对象。这个对象必须有一个名为“错误”的名称/值对。该值必须是JSON对象。

这个对象必须包含名称/值对，名称为“ code”和“ message”，并且可以包含名称/值对，名称为“ target”，“ details”和“ innererror”。

“代码”名称/值对的值是与语言无关的字符串。它的值是服务定义的错误代码，应易于阅读。与响应中指定的HTTP错误代码相比，此代码可作为错误的更具体指示。
服务应该具有相对少量（大约20个）的“代码”可能值，并且所有客户端都必须能够处理所有这些值。大多数服务将需要更多数量的更特定的错误代码，
这对于所有客户端而言都不是很有趣。如下所述，这些错误代码应在“内部错误”名称/值对中公开。为现有客户端可见的“代码”引入新值是一项重大更改，
需要增加版本。服务可以通过将新的错误代码添加到“

“消息”名称/值对的值必须是人类可读的错误表示。它旨在帮助开发人员，不适合最终用户使用。
希望向最终用户公开合适消息的服务必须通过[注释](http://docs.oasis-open.org/odata/odata-json-format/v4.0/os/odata-json-format-v4.0-os.html#_Instance_Annotations)或自定义属性来实现。
服务不应该为最终用户本地化“消息”，因为这样做可能会使值记录给可能正在记录该值的应用程序开发人员不可读，并使该值在Internet上的可搜索性降低。

“目标”名称/值对的值是特定错误的目标（例如，错误属性的名称）。

如上所述，“详细信息”名称/值对的值必须是JSON对象数组，该对象必须包含“代码”和“消息”的名称/值对，并且可以包含“目标”的名称/值对，
如上所述。“详细信息”数组中的对象通常表示在请求期间发生的不同的，相关的错误。请参见下面的示例。

“内部错误”名称/值对的值必须是一个对象。该对象的内容是服务定义的。希望返回比根级代码更具体的错误的服务必须这样做，
方法是包括“代码”的名称/值对和嵌套的 `innererror` 。每个嵌套的 `innererror` 对象都比其父对象表示更高级别的详细信息。
在评估错误时，客户必须遍历所有嵌套的“内部错误”，并选择他们了解的最深层的错误。这种方案允许服务在层次结构中的任何位置引入新的错误代码，
而不会破坏向后兼容性，只要仍然出现旧的错误代码即可。服务可以向不同的调用者返回不同级别的深度和细节。
例如，在开发环境中，最深的“ 具有自定义服务器定义属性的错误类型应在服务的元数据文档中声明。请参见下面的示例。
具有自定义服务器定义属性的错误类型应在服务的元数据文档中声明。请参见下面的示例。

错误响应可能在其任何JSON对象中包含[注释](http://docs.oasis-open.org/odata/odata-json-format/v4.0/os/odata-json-format-v4.0-os.html#_Instance_Annotations)。

我们建议，对于可能重发的任何暂时性错误，服务应包括一个Retry-After HTTP标头，该标头指示客户端在再次尝试操作之前应等待的最小秒数。

##### ErrorResponse : Object

| 属性    | 类型  | 需要 | 描述       |
| ------- | ----- | ---- | ---------- |
| `error` | Error | ✔    | 错误对象。 |

##### Error : Object

| 属性         | 类型                | 需要 | 描述                                                 |
| ------------ | ------------------- | ---- | ---------------------------------------------------- |
| `code`       | String (enumerated) | ✔    | 服务器定义的一组错误代码之一。                       |
| `message`    | String              | ✔    | 错误的人类可读表示。                                 |
| `target`     | String              |      | 错误的目标。                                         |
| `details`    | Error[]             |      | 有关导致此报告错误的特定错误的详细信息数组。         |
| `innererror` | InnerError          |      | 一个对象，它包含比当前对象更多有关该错误的特定信息。 |

##### InnerError : Object

| 属性         | 类型       | 需要 | 描述                                                 |
| ------------ | ---------- | ---- | ---------------------------------------------------- |
| `code`       | String     |      | 比包含错误提供的错误代码更具体的错误代码。           |
| `innererror` | InnerError |      | 一个对象，它包含比当前对象更多有关该错误的特定信息。 |

##### 例子

“内部错误”示例：

```json
{
  "error": {
    "code": "BadArgument",
    "message": "Previous passwords may not be reused",
    "target": "password",
    "innererror": {
      "code": "PasswordError",
      "innererror": {
        "code": "PasswordDoesNotMeetPolicy",
        "minLength": "6",
        "maxLength": "64",
        "characterTypes": ["lowerCase","upperCase","number","symbol"],
        "minDistinctCharacterTypes": "2",
        "innererror": {
          "code": "PasswordReuseNotAllowed"
        }
      }
    }
  }
}
```

在此示例中，最基本的错误代码是“ BadArgument”，但是对于感兴趣的客户端，“ innererror”中有更具体的错误代码。
该服务可能在以后的日期中添加了“ PasswordReuseNotAllowed”代码，以前仅返回了“ PasswordDoesNotMeetPolicy”。
添加新的错误代码时，现有客户端不会中断，但是新客户端可以利用它。“ PasswordDoesNotMeetPolicy”错误还包括其他名称/值对，
允许客户端确定服务器的配置，以编程方式验证用户的输入或在客户端自己的本地化消息传递中向用户显示服务器的约束。

“详细信息”示例：

```json
{
  "error": {
    "code": "BadArgument",
    "message": "Multiple errors in ContactInfo data",
    "target": "ContactInfo",
    "details": [
      {
        "code": "NullValue",
        "target": "PhoneNumber",
        "message": "Phone number must not be null"
      },
      {
        "code": "NullValue",
        "target": "LastName",
        "message": "Last name must not be null"
      },
      {
        "code": "MalformedValue",
        "target": "Address",
        "message": "Address is not valid"
      }
    ]
  }
}
```

在此示例中，请求存在多个问题，每个错误均在“详细信息”中列出。

### 7.11 HTTP状态码

应该使用标准的HTTP状态代码；有关更多信息，请参见HTTP状态代码定义。

### 7.12客户端库可选

开发人员必须能够在多种平台和语言上进行开发，例如Windows，MacOS，Linux，C＃，Python，Node.js和Ruby。

应该能够从简单的HTTP工具（例如curl）访问服务，而无需花费很多精力。

服务开发人员门户应该提供“获取开发人员令牌”的等效功能，以促进实验和curl支持。

## 8 CORS

符合Microsoft REST API准则的服务必须支持[CORS（跨源资源共享）](http://www.w3.org/TR/access-control/)。服务应支持允许的 `CORS` 来源，
并通过有效的OAuth令牌强制执行授权。服务不应通过原始验证支持用户凭证。特殊情况可能会有例外。

### 8.1客户指导

Web开发人员通常不需要做任何特殊的事情就可以利用CORS。所有握手步骤都作为它们进行的标准XMLHttpRequest调用的一部分而无形地发生。

.NET等许多其他平台都集成了对CORS的支持。

#### 8.1.1避免飞行前

由于CORS协议可以触发预检请求，这些预检请求会增加服务器的往返行程，因此对性能要求较高的应用可能会希望避免这种情况。
CORS的精神是避免对旧的不具备CORS功能的浏览器能够进行的任何简单的跨域请求进行预检。所有其他请求都需要进行飞行前检查。

请求是“简单的”，如果其方法是GET，HEAD或POST，并且除了 `Accept` ， `Accept-Language` 和 `Content-Language` 之外不包含任何请求标头，
则避免预检。对于POST请求，还允许使用 `Content-Type` 标头，
但前提是其值是 `application/x-www-form-urlencoded` ， `multipart/form-data` 或 `text/plain` 。对于任何其他标题或值，将发生预检请求。

### 8.2服务指南

至少，服务必须：

- 了解浏览器在跨域请求中发送的Origin请求标头，以及在检查访问的预检OPTIONS请求中发送的Access-Control-Request-Method请求标头。
- 如果请求中存在Origin头：
  - 如果请求使用OPTIONS方法并包含Access-Control-Request-Method标头，则它是预检请求，旨在在实际请求之前探测访问。
    否则，这是一个实际的请求。对于预检请求，除了执行以下添加标头的步骤之外，服务务必不要执行任何其他处理，并且必须返回200 OK。
    对于非预检请求，除了请求的常规处理之外，还会添加以下标头。
  - 将Access-Control-Allow-Origin标头添加到响应中，该标头包含与Origin请求标头相同的值。请注意，这需要服务动态生成标头值。
    不需要cookie或任何其他形式的[用户凭证的资源](http://www.w3.org/TR/access-control/#user-credentials)可以使用通配符星号（*）代替。
    请注意，通配符仅在此处可接受，不适用于以下所述的任何其他标头。
  - 如果调用方需要访问不在[简单响应标头](http://www.w3.org/TR/access-control/#simple-header)集中的[响应标头](http://www.w3.org/TR/access-control/#simple-header)
    （缓存控制，内容语言，内容类型，过期，最后修改，实用），则添加访问控制暴露-标头标头，包含客户端应有权访问的其他响应标头名称的列表。
  - 如果请求需要cookie，则添加一个Access-Control-Allow-Credentials标头设置为“ true”。
  - 如果该请求是预检请求（请参阅第一个项目符号），则该服务必须：
    - 添加一个 `Access-Control-Allow-Headers` 响应标头，其中包含允许客户端使用的请求标头名称的列表。
      该列表仅需要包含不在[简单请求标头](http://www.w3.org/TR/access-control/#simple-header)集中的[标头](http://www.w3.org/TR/access-control/#simple-header)
      （Accept，Accept-Language，Content-Language）。如果服务接受的头没有任何限制，则该服务可以简单地返回与客户端发送的Access-Control-Request-Headers头相同的值。
    - 添加一个Access-Control-Allow-Methods响应标头，其中包含允许调用方使用的HTTP方法的列表。

添加一个 `Access-Control-Max-Age` 首选项响应标头，其中包含该首选项响应有效的秒数（因此可以在随后的实际请求之前避免使用）。
请注意，虽然习惯使用较大的值，如2592000（30天），但许多浏览器会自行施加一个较低的限制（例如5分钟）。

由于浏览器的飞行前响应缓存非常差，因此飞行前响应带来的额外往返行程会影响性能。性能至关重要的交互式Web客户端使用的服务应避免导致预检请求的模式

- 对于GET和HEAD调用，请避免要求不属于上述简单设置的请求标头。允许将它们作为查询参数提供。
  - Authorization标头不是简单集的一部分，因此对于需要身份验证的资源，必须通过“ access_token”查询参数发送身份验证令牌。
    请注意，不建议在URL中传递身份验证令牌，因为它可能导致令牌记录在服务器日志中并暴露给有权访问这些日志的任何人。
    通过URL接受身份验证令牌的服务必须采取措施来减轻安全风险，例如使用短暂的身份验证令牌，禁止身份验证令牌被记录以及控制对服务器日志的访问。
- 避免要求Cookie。如果设置了“ withCredentials”属性，则XmlHttpRequest仅在跨域请求上发送cookie。这也会导致预检请求。
  - 需要基于Cookie的身份验证的服务务必使用“动态Canary”来保护所有接受Cookie的API。
- 对于POST调用，请在适用的情况下（“应用程序/ x-www-form-urlencoded”，“ multipart / form-data”，“ text / plain”）集中使用简单的Content-Type。任何其他Content-Type都会引发预检请求。
  - 服务不得以避免CORS飞行前请求的名义违反其他API建议。特别是，根据建议，由于内容类型，大多数POST请求实际上将需要进行预检请求。
  - 如果消除预检是至关重要的，则服务可以支持替代的数据传输机制，但是必须也支持推荐的方法。

另外，在适当的服务时，可以支持JSONP模式，以进行简单的，仅限GET的跨域访问。在JSONP中，服务采用指示格式（*$ format = json*）的参数
和指示回调（*$ callback = someFunc*）的参数，并返回文本/ javascript文档，该文档包含包装在具有指定名称的函数调用中的JSON响应。
Wikipedia上有关JSONP的更多信息：[JSONP](https://en.wikipedia.org/wiki/JSONP)。

## 12 版本

**所有符合Microsoft REST API准则的API都必须支持显式版本控制。**客户端可以依靠服务保持稳定是至关重要的，服务可以添加功能并进行更改也至关重要。

### 12.1版本格式

使用Major.Minor版本控制方案对服务进行版本控制。服务可以选择“仅主要”版本方案，在这种情况下暗含“ .0”，并且本节中的所有其他规则均适用。支持两种用于指定REST API请求版本的选项：

- 嵌入在请求URL的路径中，位于服务根目录的末尾： `https://api.contoso.com/v1.0/products/users`
- 作为URL的查询字符串参数： `https://api.contoso.com/products/users?api-version=1.0`

在这两个选项之间进行选择的指导如下：

1. 位于DNS端点后面的服务必须使用相同的版本控制机制。
2. 在这种情况下，跨端点的一致用户体验至关重要。Microsoft REST API准则工作组建议在未与组织领导团队进行明确对话的情况下，不要创建新的顶级DNS终结点。
3. 保证其REST API的URL路径稳定的服务，即使通过API的未来版本，也可以采用查询字符串参数机制。这意味着在API交付后，API中描述的关系的命名和结构就无法发展，即使是在具有重大更改的版本之间也是如此。
4. 无法确保未来版本中URL路径稳定性的服务必须将版本嵌入URL路径中。

#### 12.1.1组版本控制

组版本控制是一项可选功能，可以使用查询字符串参数机制在服务上提供。组版本允许在通用版本名称下对API端点进行逻辑分组。这使开发人员可以查找单个版本号，并在多个端点之间使用它。组版本号是众所周知的，服务应该拒绝任何无法识别的值。

在内部，服务将采用组版本并将其映射到适当的Major.Minor版本。

组版本格式定义为YYYY-MM-DD，例如2012年12月7日为2012-12-07。此日期版本格式仅适用于组版本，不应用作替代Major.Minor版本。

##### 12.1.1.1组版本控制的示例

```text
Group      | Major.Minor
---------- | -----------
2012-12-01 | 1.0
           | 1.1
           | 1.2
2013-03-21 | 1.0
           | 2.0
           | 3.0
           | 3.1
           | 3.2
           | 3.3
```

| 版本格式                       | 例                     | 解释     |
| ------------------------------ | ---------------------- | -------- |
| {groupVersion}                 | 2013-03-21，2012-12-01 | 3.3、1.2 |
| {majorVersion}                 | 3                      | 3.0      |
| {majorVersion}。{minorVersion} | 1.2                    | 1.2      |

客户端可以指定组版本或专业版本。

例如：

```test
GET http://api.contoso.com/acct1/c1/blob2?api-version=1.0
PUT http://api.contoso.com/acct1/c1/b2?api-version=2011-12-07
```

### 12.2何时版本

服务必须响应任何重大的API更改而增加其版本号。请参阅以下部分，详细讨论什么是重大更改。如果需要，服务也可以增加其版本号以进行不间断的更改。

使用新的主要版本号表示将来将不再支持现有客户端。引入新的主要版本时，服务必须为现有客户提供清晰的升级路径，并制定与业务组策略一致的弃用计划。服务应将新的次要版本号用于所有其他更改。

版本化服务的在线文档务必指出每个先前API版本的当前支持状态，并提供最新版本的路径。

### 12.3重大更改的定义

对API合同的更改被视为重大更改。影响API向后兼容性的更改是一项重大更改。

团队可以根据他们的业务需求定义向后兼容性。例如，Azure将响应中的新JSON字段定义为不向后兼容。Office 365具有向后兼容性的较宽松定义，并允许将JSON字段添加到响应中。

重大更改的清晰示例：

1. 删除或重命名API或API参数
2. 现有API的行为更改
3. 错误代码和故障合同的变更
4. 任何违反[最小惊讶原则的](http://en.wikipedia.org/wiki/Principle_of_least_astonishment)东西

服务必须明确定义其重大更改的定义，尤其是在向JSON响应中添加新字段以及使用默认字段添加新API参数方面。与其他服务一起位于DNS端点后面的服务在定义合同可扩展性时必须保持一致。

[OData V4规范本节中](http://docs.oasis-open.org/odata/odata/v4.0/errata02/os/complete/part1-protocol/odata-v4.0-errata02-os-part1-protocol-complete.html#_Toc406398209)描述的适用更改应被视为所有服务必须考虑的重大更改的最低要求的一部分。

## 13 长期支持

长时间运行的操作（有时称为异步操作）对不同的人意味着不同的意思。本节针对不同类型的长时间运行制定了指南，并介绍了这些类型的运行的线路协议和最佳实践。

1. 一个或多个客户端必须能够同时监视和操作同一资源。
2. 系统状态应始终可发现和可测试。即使操作跟踪资源不再处于活动状态，客户端也应该能够确定系统状态。查询长时间运行状态的操作本身应该利用网络原理。即具有统一接口语义的定义良好的资源。客户端可以在某些资源上发出GET来确定长时间运行的状态
3. 长期运行的操作应该为希望“解雇”的客户以及希望积极监控结果并根据结果采取行动的客户服务。
4. 取消并不明确意味着回滚。在每个API定义的情况下，这可能意味着回滚，补偿，完成或部分完成等。取消操作之后，客户不应将服务恢复到允许继续服务的一致状态的责任。

### 13.1基于资源的长期运行（RELO）

基于资源的建模是将操作的状态编码在资源中，并且所使用的有线协议是标准同步协议。在此模型中，状态转移定义明确，目标状态定义类似。

*对于长时间运行的操作，这是首选模型，应尽可能使用它。*避免了LRO Wire Protocol的复杂性和机制，对于我们的用户和工具链而言，事情变得更加简单。

一个示例可能是计算机重新引导，该操作本身同步完成，但是虚拟机资源上的GET操作将具有“状态：正在重新引导”，“状态：正在运行”，可以随时查询。

该模型可以集成推送通知。

虽然大多数操作可能是POST语义，但除POST语义服务外，MAY还可以通过路由支持PUT语义，以简化其API。例如，想要创建一个名为“ db1”的数据库的用户可以调用：

```test
PUT https://api.contoso.com/v1.0/databases/db1
```

在这种情况下，数据库段正在处理PUT操作。

服务也可以使用下面定义的混合。

### 13.2分步长时间运行

逐步操作是一种需要很长且通常是不可预测的时间来完成的操作，并且不提供在资源中建模的状态转换。本节概述了服务应使用的方法来公开这些长时间运行的操作。

服务可以公开逐步操作。

> 逐步长时间运行的操作有时称为“异步”操作。这会造成混乱，因为它将平台的元素（“异步/等待”，“承诺”，“未来”）与API操作的元素混合在一起。本文档使用术语“逐步运行”或经常仅使用“逐步操作”以避免混淆“异步”一词。

服务必须对逐步请求执行尽可能多的同步验证。服务必须以同步方式确定返回错误的优先级，目标是使用长时间运行的操作有线协议仅处理“有效”操作。

对于定义为逐步长期运行操作的API，服务必须经过逐步长期运行操作流程，即使该操作可以立即完成。换句话说，API必须采用并坚持LRO模式，并且不得根据情况更改模式。

#### 13.2.1 PUT

服务可以启用用于实体创建的PUT请求。

```test
PUT https://api.contoso.com/v1.0/databases/db1
```

在这种情况下，*数据库*段正在处理PUT操作。

```test
HTTP/1.1 202 Accepted
Operation-Location: https://api.contoso.com/v1.0/operations/123
```

对于需要返回此处创建201的服务，请使用下面描述的混合流程。

202已接受不返回任何正文。201 Created案例应返回目标资源的主体。

#### 13.2.2 POST

服务可以启用用于实体创建的POST请求。

```test
POST https://api.contoso.com/v1.0/databases/

{
  "fileName": "someFile.db",
  "color": "red"
}
```

```test
HTTP/1.1 202 Accepted
Operation-Location: https://api.contoso.com/v1.0/operations/123
```

#### 13.2.3 POST，混合模型

服务可以对创建资源的集合的POST请求进行同步响应，即使在生成响应时并未完全创建资源。为了使用这种模式，响应必须包括不完整资源的表示和不完整资源的指示。

例如：

```test
POST https://api.contoso.com/v1.0/databases/ HTTP/1.1
Host: api.contoso.com
Content-Type: application/json
Accept: application/json

{
  "fileName": "someFile.db",
  "color": "red"
}
```

服务响应说数据库已经创建，但是通过包含Operation-Location标头指示请求未完成。在这种情况下，响应有效负载中的状态属性还指示操作尚未完全完成。

```test
HTTP/1.1 201 Created
Location: https://api.contoso.com/v1.0/databases/db1
Operation-Location: https://api.contoso.com/v1.0/operations/123

{
  "databaseName": "db1",
  "color": "red",
  "Status": "Provisioning",
  [ … other fields for "database" …]
}
```

#### 13.2.4 操作资源

服务可以在租户级别提供“ / operations”资源。

提供“ /operations”资源的服务必须提供GET语义。GET必须枚举遵循标准分页，排序和过滤语义的操作集。此操作的默认排序顺序必须为：

| 主要排序     | 次要排序     |
| ------------ | ------------ |
| 未开始的操作 | 操作创建时间 |
| 运作中       | 操作创建时间 |
| 完成的操作   |              |

请注意，“完成的操作”是目标状态（请参见下文），实际上可以是几个不同的状态中的任何一个，例如“成功”，“已取消”，“失败”等。

#### 13.2.5操作资源

操作是跟踪逐步运行的长期操作的用户可访问资源。操作必须支持GET语义。针对某个操作的GET操作务必返回：

1. 操作资源，其状态以及与特定API相关的任何扩展状态。
2. 200 OK作为响应代码。

服务可以通过在操作上暴露DELETE来支持取消操作。如果支持，则DELETE操作必须是幂等的。

> 注意：从API设计的角度来看，取消并不明确意味着回滚。在按API定义的情况下，这可能意味着回滚，补偿或完成或部分完成等。在取消操作之后，客户不应将服务返回到允许继续提供服务的一致状态的责任。

不支持取消操作的服务必须在发生DELETE时返回405 Method Not Allowed。

操作必须支持以下状态：

1. 没有开始
2. 运行
3. 成功。终端状态。
4. 失败了 终端状态。

服务可以添加其他状态，例如“已取消”或“部分完成”。支持取消的服务必须充分描述其取消，以便可以准确地确定系统状态并可以运行任何补偿操作。

支持其他状态的服务应考虑以下规范名称列表，并尽可能避免创建新名称：取消，取消，中止，中止，墓碑，删除，删除。

一个操作必须包含以下信息，并在GET响应中提供以下信息：

1. 创建操作的时间戳。
2. 当前状态输入的时间戳。
3. 操作状态（未启动/正在运行/已完成）。

服务可以在操作中添加其他特定于API的字段。返回的操作状态JSON如下所示：

```test
{
  "createdDateTime": "2015-06-19T12-01-03.45Z",
  "lastActionDateTime": "2015-06-19T12-01-03.45Z",
  "status": "notstarted | running | succeeded | failed"
}
```

##### 13.2.5.1完成百分比

有时，服务无法完全准确地知道操作何时完成。这使得使用Retry-After标头有问题。在这种情况下，服务可以在operationStatus JSON中包含完成百分比字段。

```test
{
   “ createdDateTime ”：“ 2015-06-19T12-01-03.45Z ”，
   “ percentComplete ”：“ 50 ”，
   “ status ”：“ running ” 
}
```

在此示例中，服务器已向客户端指示长时间运行的操作已完成50％。

##### 13.2.5.2目标资源位置

对于产生或操纵资源的操作，服务必须在操作完成后的状态中包含目标资源位置。

```test
{
  "createdDateTime": "2015-06-19T12-01-03.45Z",
  "lastActionDateTime": "2015-06-19T12-06-03.0024Z",
  "status": "succeeded",
  "resourceLocation": "https://api.contoso.com/v1.0/databases/db1"
}
```

#### 13.2.6删除操作

服务可以选择支持逻辑删除操作。服务可以在服务定义的一段时间后选择删除逻辑删除。

#### 13.2.7典型流程，轮询

- 客户端通过使用POST调用动作来调用逐步操作
- 服务器必须通过响应202接受的状态码来指示请求已开始。响应应包含位置标头，该标头包含一个URL，客户端应等待Retry-After标头中指定的秒数后，客户端应轮询结果。
- 客户端轮询该位置，直到从服务器收到200 OK响应。
