---
title: API 设计指南
author: kevin
date: 2020-08-21 14:22:00
update: 2021-05-19 19:15:00
tags:
- api-desig
---

在当前微服务大环境之下，越来越多的项目使用 RESTful HTTP接口方位资源 。为了团队开发的友好体验，开发人员尽可能遵循一致的 REST API 设计原则就显得至关重要。

API 设计指南旨在提供一套基础规范，便于团队设计和开发出一致的 RESTful 接口。

<!-- more -->

## 2. 目录

[TOC]

## 3. 引言

在各个子系统间，开发人员基本都是通过 HTTP 接口访问。尽管服务使用不同语言开发，但最终都是 HTTP 请求。为了能开发出来的产品能够支持更广泛的客户端和服务，并且不会依赖特定开发框架或者环境，就需要在 API 上有良好的设计。这些规范的目的就是尽可能让所有 HTTP 客户端能轻松、一致地使用 REST API。

在吸取了社区中大量成熟的经验后，制定出了此规范。主要参考规范如下：

- [微软 REST API 规范](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md)
- [PayPal REST API 规范](https://github.com/paypal/api-standards/blob/master/api-style-guide.md)

### 3.1 推荐阅读

为了开发良好 HTTP 服务，建议理解 REST 体系结构风格的原理。如果您不熟悉 RESTful 设计，可以参考一下资源：

[Wikipedia 上的 REST](http://en.wikipedia.org/wiki/Representational_state_transfer) - REST 背后的常见定义和核心思想概述。

[REST论文](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm) —— Roy Fielding 论文中有关 REST 的一章，即“架构风格和基于网络的软件架构设计” 。

[RFC 7231](https://tools.ietf.org/html/rfc7231) —— HTTP/1.1 规范。

[REST 实践](http://www.amazon.com/REST-Practice-Hypermedia-Systems-Architecture/dp/0596805829/) —— 关于 REST 基础的书籍。

## 4. 名词释义

本节内容主要阐述文档中的特殊关键词的含义，界定其范围和使用场景，避免因对关键词理解偏差而造成使用不当。

### 4.1 资源

资源是 REST 中信息的关键抽象。根据 [Fielding 的论文 5.2 节](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2)，可以命名的任何信息都可以是资源，
如：文档或图像，临时服务（例如“洛杉矶今天的天气”），其他资源的集合，非虚拟对象（例如一个人），等等。资源是到一组实体的概念性映射，而不是在任何特定时间点与该映射相对应的实体。
更准确地说，资源 `R` 是随时间变化的隶属函数 `MR(t)` ，用于将时间 `t` 映射到等效的一组实体或值。集合中的值可以是资源表示和 `/` 或资源标识符。

资源也可以映射到空集，从而允许在存在任何概念之前就对该概念进行引用。

### 4.2 资源标识符

REST 使用资源标识符来标识组件之间交互中涉及的特定资源实例。命名机构（例如提供 API 的组织）分配了资源标识符以使其有可能引用资源，
它负责维护映射随时间的语义有效性（确保成员资格函数不变）。- [菲尔丁的论文第 5.2 节](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2)

### 4.3 域

根据 Wikipedia 定义，[领域模型](https://en.wikipedia.org/wiki/Domain_model)是一个抽象系统，描述了知识，影响或活动领域的选定方面。这些概念包括业务中涉及的数据，
以及业务针对该数据使用的规则。例如， PayPal 域模型包括诸如支付，风险，合规性，身份，客户支持等域。

### 4.4 功能

功能是面向业务和面向客户所组织的业务逻辑视图。功能可用于将 API 打包组合为系统的稳定的、业务驱动的视图，供客户和体验使用。功能的例子包括：合规性，信用，身份，零售和风险等。

功能驱动接口，而域粒度更粗，更接近代码和组织的结构。从服务的角度来看，功能和领域被视为正交的关注点。

### 4.5 命名空间

功能驱动 API 组合中的服务建模和名称空间问题。名称空间是业务功能模型的一部分。名称空间的例子有:合规性、设备、传输、信用、限制等。。

命名空间应反映逻辑上将一组业务功能分组的域。域定义应反映客户对平台功能组织方式的看法。请注意，这些可能不一定反映公司的层次结构，组织或（现有）代码结构。
在某些情况下，从定义上可以反映目标，面向客户的平台组织模型的意义上说，领域定义是理想的。底层服务实现和组织结构可能需要迁移以反映这些边界。

### 4.6 服务

服务是访问和操作资源值集的通用 API ，而不管成员功能是如何定义的，也不管处理请求的软件类型是什么。服务是软件的通用部分，可以执行任意数量的功能。因此，考虑存在的不同类型的服务是有益的。

从逻辑上讲，我们可以将公开的服务和 API 分为两类：

1. **功能 API** 是实现通用，可重用业务功能的服务所公开的公共API。
2. **特定场景的 API** 建立在功能 API 之上，并公开可能针对特定问题的功能，或针对通用功能的特定于上下文而专门优化的功能。上下文信息可能和时间，位置，设备，通道，身份，用户，角色，授权级别等有关。

#### 4.6.1 基于功能的服务和 API

功能 API 是可重用业务功能的公共接口。*公开* 指这些 API 仅限于供前端使用，外部使用者或来自不同域的内部使用者使用的接口。

#### 4.6.2 特定场景的服务和 API

特定场景的服务在核心功能上提供了最少的附加业务逻辑，并且主要提供了转换和轻量编排，以根据特定场景，渠道或设备的需求量身定制交互。它们的输入/输出仅限于服务调用。

### 4.7 客户端，API 客户端，API 使用者

调用 API 请求并使用 API 响应的实体。

### 4.8 幂等(Idempotency)

幂等性是构建容错 API 的一个重要方面。幂等 API 使客户机能够安全地重试操作，而不必担心操作可能造成的副作用。例如，在请求由于网络连接错误而失败的情况下，客户机可以安全地重试幂等请求。

根据 [HTTP 规范](https://tools.ietf.org/html/rfc2616#section-9.1.2)，如果一个以上相同请求的副作用与单个请求的副作用相同，则该方法是幂等的。方法 `GET`， `HEAD` ， `PUT` 和 `DELETE` （另外， `TRACE` 和 `OPTIONS` ）是幂等的。

根据定义， `POST` 操作既不安全也不幂等。

所有服务实现都必须确保根据 HTTP 规范实现 HTTP 方法的安全和幂等行为。`POST` 操作需要幂等的服务必须按照以下准则实施。

#### 4.8.1 POST请求的幂等性

根据定义， `POST` 操作不是幂等的，这意味着使用相同的输入多次执行 `POST` 操作会创建相同数量的资源。为了避免创建重复的资源， API 应该实现下面一节中定义的协议。这保证了为相同的输入内容只创建一条记录。

对于那些需要 `POST` 请求具有幂等性的用例，创建重复记录是一个严重的问题。例如，不允许在账户上创建或执行重复的付款记录。

为了跟踪幂等请求，在每个请求中发送唯一的**幂等密钥**。定义 Header ，并将其值用作每个请求的**幂等密钥**。

**例子：**

客户端：

API 客户端发送带有 `Foo-Request-Id` 标头的 `POST` 请求，该标头包含**幂等密钥**。

```none
POST /v1/payments/referenced-payouts-items HTTP/1.1
Host: api.foo.com
Content-Type: application/json
Authorization: Bearer oauth2_token
Foo-Request-Id: 123e4567-e89b-12d3-a456-426655440000

{
    "referenceIdd": "4766687568468",
    "referenceType": "egflf465vbk7468mvnb"
}
```

服务端：

如果调用成功并创建资源，则服务必须返回 `201` 响应，表示成功和状态改变。

响应：

```none
HTTP/1.1 201 CREATED
Content-Type: application/json

{

  "itemId": "CDZEC5MJ8R5HY",
  "links": [{
      "href": "https://api.foo.com/v1/payments/referenced-payouts-items/CDZEC5MJ8R5HY",
      "rel": "self",
      "method": "GET"
  }]
}
```

服务的响应中包含带有**幂等密钥**的 `Foo-Request-Id` 。

**对于来自客户端的具有相同输入有效内容的后续请求：**

客户端：

API 客户端发送的 `POST` 请求具有与以前相同的**幂等密钥**和请求体。

```none
POST /v1/payments/referenced-payouts-items HTTP/1.1
Host: api.foo.com
Content-Type: application/json
Authorization: Bearer oauth2_token
Foo-Request-Id: 123e4567-e89b-12d3-a456-426655440000


{
    "reference_id": "4766687568468",
    "reference_type": "egflf465vbk7468mvnb"
}
```

服务器在确认调用与第一次执行相同后，必须返回 `200` 响应，并带有资源信息，以表明请求已被成功处理。

```none
HTTP/1.1 200 OK
Content-Type: application/json

{
    "item_id": "CDZEC5MJ8R5HY",
    "processing_state": {
        "status": "PROCESSING"
    },
    "reference_id": "4766687568468",
    "reference_type": "egflf465vbk7468mvnb",
    "payout_amount": {
        "currency_code": "USD",
        "value": "2.0"
    }
    "payout_destination": "9C8SEAESMWFKA",
    "payout_transaction_id": "35257aef-54f7-43cf-a258-3b45caf3293",
    "links": [{
        "href": "https://api.foo.com/v1/payments/referenced-payouts-items/CDZEC5MJ8R5HY",
        "rel": "self",
        "method": "GET"
    }]
}
```

#### 4.8.2 幂等键的唯一性

作为每个 `POST` 请求的一部分提供的**幂等密钥**必须是惟一的，不能在具有不同请求体的其他请求中重用。请参阅下面描述的错误场景，以了解在使用重复**幂等密钥**的请求下，服务器的行为。

如何使密钥唯一取决于客户机以及它与服务器达成的协议。建议使用 `UUID` 或类似的随机标识符作为**幂等密钥**。还建议服务器实现基于时间的幂等键，从而能够在密钥到期时清除或删除它。

错误场景：

- 如果幂等请求缺少 `Foo-Request-Id` 标头，则该服务必须返回 `400` 错误，并带有指向有关此模式的公共文档的链接。
- 如果试图用其他请求有效数据重用幂等密钥，则该服务必须返回 `422` 错误，并带有指向有关此模式的公共文档的链接。
- 对于其他错误，服务必须返回适当的错误消息。

## 5. 设计原则

本节阐述服务设计的原则，这些服务是向内部和外部开发人员，部门间，合作伙伴和关联公司公开 API 。服务是指与特定场景有关的功能，以 API 形式公开。

以下是服务的核心设计原则。

### 5.1 松耦合

**服务和消费者必须彼此松散耦合。**

耦合是指两件事之间的联系或关系。耦合程度相当于依赖的级别。该原则提倡服务契约的设计，并始终强调减少（**松散**）服务契约、与其实施和服务使用者之间的依赖性。

松散耦合的原则促进了服务逻辑和实现的独立设计和发展，同时仍然强调了与已经依赖该服务功能的消费者之间的基本互操作性。

该原则意味着以下几点：

- 服务契约不应公开实施细节
- 服务契约可以演化而不会影响现有消费者
- 特定域中的服务可以独立于其他域演化

### 5.2 封装形式

**域服务只能通过其他服务契约访问其没有的数据和功能。**

服务公开其拥有和实现的功能和数据，以及不依赖于它的功能和数据。该原则主张，任何服务所依赖且不属于它的功能或数据都必须仅通过服务契约来访问。

该原则意味着以下几点：

- 服务具有明确的隔离边界
- 在功能和数据方面明确的所有权范围
- 服务不能直接公开它不拥有的数据

### 5.3 稳定性

**服务契约必须稳定。**

服务的设计方式必须让它暴露的契约对现有客户仍然有效。如果服务契约需要以与消费者不兼容的方式发展，则应清楚地传达这一点。

该原则意味着以下几点：

- 现有的客户服务必须在记录的时间内得到支持
- 必须以不影响现有消费者的方式引入其他功能
- 必须清楚地说明弃用和迁移政策，让消费者有心里预期

### 5.4 可重用

**服务必须开发为可在多个上下文中被多个使用者重用。**

API 平台的主要目标是通过使用和组合服务来使应用程序能够快速，经济高效地开发。只有在为多个用例和多个消费者灵活开发服务契约时，这才可能实现。该原则主张服务的开发方式应使其能够被多个消费者和在多种环境中使用，其中某些环境可能会随着时间而发展。

该原则意味着以下几点：

- 服务契约不仅应针对当前环境而设计，还应具有支持和/或可扩展，以供多个消费者在不同环境中使用
- 服务契约可能需要随着时间的推移逐步发展以支持多种环境和消费者

### 5.5 基于契约

**功能和数据只能通过标准化服务契约公开。**

服务通过服务契约公开其用途和功能。服务契约包括功能方面，非功能方面（例如可用性，响应时间）和业务方面（例如每次通话费用，条款和条件）。*标准化* 意味着服务契约必须符合契约设计标准。

该原则主张所有功能和数据都只能通过标准化服务契约公开。因此，服务的消费者只能通过服务契约来理解和访问功能和数据。

该原则意味着以下几点：

- 在服务契约之外无法理解或访问功能和数据
- 每条数据（例如在数据存储中管理的数据）只属于一个服务

### 5.6 一致性

**服务必须遵循一组通用的规则，交互方式，词汇和共享类型。**

通过一组规则，可以定义一致的公开服务。通过减少使用者对新服务的学习曲线，来提高了 API 平台的易用性。

该原则意味着以下几点：

- 为服务规定了一套标准
- 服务应使用通用和共享词典中的词汇
- 兼容的交互方式，服务粒度和共享类型是实现完全互操作性和简化服务组合的关键

### 5.7 使用方便

**服务必须易于使用并在使用者（和应用程序）中组成。**

难用且耗时的服务会让消费者寻找其他机制来访问相同功能，这会降低微服务所带来的收益。可组合性意味着可以轻松组合服务，因为服务契约和访问协议是一致的，并且不必对每个服务契约有不同的理解。

该原则意味着以下几点：

- 服务契约易于发现和理解
- 服务契约和协议在所有方面都是一致的，例如，标识和认证机制，错误语义，通用类型用法，分页等。
- 服务具有明确的所有权，因此消费者提供者可以就 SLA ，要求和问题与服务所有者联系
- 服务商可以轻松集成，测试和部署使用
- 服务商可以轻松地监视服务的非功能性指标

### 5.8 可外部化

**服务的设计必须使其提供的功能易于外部化。**

服务是为来自另一个领域或团队、另一个业务单元或另一个公司的消费者开发的。在所有这些情况下，暴露的功能是相同的；更改的是访问机制或服务执行的策略，如身份验证、授权和速率限制。由于公开的功能是相同的，因此服务应该设计一次，然后根据业务需要通过适当的策略进行外部化。

该原则意味着以下几点：

- 服务接口必须从域模型及它支持的用例中得出
- 支持的服务契约和访问（绑定）协议必须满足消费者的需求
- 服务的外部化不得要求重新实现或更改服务契约

## 6. HTTP 规范

本节阐述 HTTP 规范，并根据通用规范做出部分限定条件，以达到风格统一。

### ６.1 URL

#### 6.1.1 URL 结构

设计的 URL 应该简单和阅读理解，而不是让人感到困惑或者造成理解错误。在没有较好的客户端库支持的情况下，设计良好的 URL 有助于在多平台上使用。

设计良好的 URL ：

```bash
https://api.contoso.com/v1.0/people/jdoe@contoso.com/inbox
```

设计不好 URL ：

```bash
https://api.contoso.com/EWS/OData/Users('jdoe@microsoft.com')/Folders('AAMkADdiYzI1MjUzLTk4MjQtNDQ1Yy05YjJkLWNlMzMzYmIzNTY0MwAuAAAAAACzMsPHYH6HQoSwfdpDx-2bAQCXhUk6PC1dS7AERFluCgBfAAABo58UAAA=')
```

使用 URL 作为参数值是常见的情况。例如，这种形式是可以接受的：

```bash
https://api.contoso.com/v1.0/items?url=https://resources.contoso.com/shoes/fancy
```

#### 6.1.2 URL长度

在 RFC 7230 的 [3.1.1](https://tools.ietf.org/html/rfc7230#section-3.1.1) 节中定义的 HTTP 1.1 消息格式在请求行上没有长度限制，该长度包括目标URL。

> HTTP 并未对请求行的长度设置预定义的限制。[...]当接收到比期望的 URL 长度更长的时候，服务器必须以 414（URI太长）状态码作为响应。

当服务的 URL 长度超过 2083 个字符的时候 ，就需要考虑虑客户端使用是否方便。可以参考如下讨论：

- `http://stackoverflow.com/a/417184`
- `https://blogs.msdn.microsoft.com/ieinternals/2014/08/13/url-length-limits/`

还要注意，在设计服务的时候，应兼顾技术栈的在 URL 上的限制。

### 6.2 HTTP 方法

HTTP 定义了一组请求方法, 以表明要对给定资源执行的操作。指示针对给定资源要执行的期望动作. 虽然他们也可以是名词, 但这些请求方法有时被称为 HTTP 动词。
每一个请求方法都实现了不同的语义, 但会有一些共同的特征。例如一个请求方法可以是 [安全](https://developer.mozilla.org/zh-CN/docs/Glossary/safe),
[幂等](https://developer.mozilla.org/zh-CN/docs/Glossary/%E5%B9%82%E7%AD%89), 或 [可缓存](https://developer.mozilla.org/en-US/docs/Glossary/cacheable).

安全的 HTTP 方法是指这是个不会修改服务器的数据的方法。也就是说，这是一个对服务器只读操作的方法。`GET`， `HEAD` 和 `OPTIONS` 都是安全的 HTTP 方法。所有安全的方法都是幂等的，而 `PUT` 和 `DELETE` 则是不安全的。

幂等值的是一个 HTTP 请求被执行一次与连续执行多次的效果是一样的，服务器的状态也是一样的。换句话说就是，幂等方法不应该具有副作用（统计用途除外）。幂等性只与后端服务器的实际状态有关，而每一次请求接收到的状态码不一定相同。

可缓存响应是一个可以被缓存的HTTP响应，它被存储起来以供以后检索和使用，并将一个新的请求保存到服务器。不是所有的HTTP响应都可以被缓存，以下是要缓存的HTTP响应的约束:

- 请求中使用的方法本身是可缓存的，即 `GET` 或 `HEAD` 方法。
- 应用程序缓存响应的状态码，并且将其视为可缓存的。下面的状态代码可缓存：`200`，`203` ， `204` ， `206` ， `300` ， `301` ， `404` ， `405` ， `410` ， `414` ，和 `501` 。
- 响应中有特定的标头，例如 `Cache-Control` ，可以控制缓存。

并非所有资源都支持这些方法，但是使用以下方法的所有资源必须符合其用法。

| 方法                                                                         | 描述                                             | 是否有请求主体| 成功响应是否有主体   | 是否安全 | 是否幂等 | 是否可缓存     |
| ---------------------------------------------------------------------------- | ------------------------------------------------ | ---------- | ------------- | ----- | ------- | ------------ |
| [GET](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/GET)         | 请求指定的资源。使用 GET 的请求应该只用于获取数据。     | 否         | 是                | 是       | 是      | 是            |
| [POST](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/POST)       | 根据提供的数据创建新对象，或提交命令 。                 | 是         | 是                | 否       | 否      | 只在有新数据是 |
| [PUT](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PUT)         | 使用请求中的数据创建或者替换目标资源。                  | 是         | 否                | 否       | 是      | 否           |
| [DELETE](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/DELETE)   | 用于删除指定的资源。                                    | 可以有     | 可以有             | 否       | 是      | 否            |
| [HEAD](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/HEAD)       | 请求资源的 Header 信息, 且 Header 与 GET 请求返回的一致，只是没有响应体。    | 否         | 否                | 是       | 是      | 是        |
| [PATCH](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PATCH)     | 用于对资源进行部分修改。                                | 是         | 否                | 否       | 否      | 否        |
| [OPTIONS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/OPTIONS) | 用于获取目的资源所支持的通信选项。                      | 否         | 是                | 是       | 是      | 否        |

#### 6.2.1 POST

`POST` 方法不是幂等的，多次调用，会执行多次结果。如果调用创建资源的接口，多次调用会创建多个资源

- `POST` 方法应该用于创建一个资源
  - 例： `POST https://api.foo.com/v1/users`
- `POST` 方法应用于创建一个子资源，并建立其与主资源间的关系
  - 例： `POST https://api.foo.com/v1/payments/12345/refund`
- `POST` 方法也可以根据名称用于复合操作

如果应用于建资源。操作成功，返回 `201 Created` 状态码。其他处理成功返回 `200 OK` 状态码

#### 6.2.2 PUT

`PUT` 使用请求中的数据创建或者替换目标资源。其是幂等的，即调用一次与连续调用多次是等价的。

处理成功返回 `204 No Content` 状态码。

#### 6.2.3 DELETE

如果 `DELETE` 方法成功执行，那么可能会有以下几种状态码：

- `200 OK` ：表示操作已执行，并且响应中提供了相关状态的描述信息。
- `202 Accepted` ：表示请求的操作可能会成功执行，但是尚未开始执行。
- `204 No Content`：表示操作已执行，但是无进一步的相关信息。

#### 6.2.4 PATCH

IETF 已将 `PATCH` 定义为用于增量更新现有对象的方法（请参阅 [RFC 5789](http://tools.ietf.org/html/rfc5789)）。符合 REST API 准则的 API 应该支持 `PATCH` 。

不同于 `PUT` 方法，而与 `POST` 方法类似，`PATCH` 方法不是幂等的，这就意味着连续多个的相同请求会产生不同的效果。

#### 6.2.5 OPTIONS

`OPTIONS` 用于获取目的资源所支持的通信选项。

通过 `OPTIONS` 返回响应头中 `Access-Control-Request-Method` 值来获悉该 URL 支持那些 HTTP 方法。

### 6.3 Header

HTTP 消息头允许客户端和服务器通过请求和响应传递附加信息。一个请求头由名称（不区分大小写）后跟一个冒号“:”，冒号后跟具体的值（不带换行符）组成。该值前面的引导空白会被忽略。

自定专用消息头可通过 `X-` 前缀来添加；但是这种用法被 IETF 在 2012 年 6 月发布的 [RFC5548](https://tools.ietf.org/html/rfc6648) 中明确弃用，
原因是其会在非标准字段成为标准时造成不便；其他的消息头在 [IANA 注册表](http://www.iana.org/assignments/message-headers/perm-headers.html)中列出，
其原始内容在 [RFC 4229](http://tools.ietf.org/html/rfc4229) 中定义。此外， IANA 还维护着[被提议的新 HTTP 消息头注册表](http://www.iana.org/assignments/message-headers/prov-headers.html)。

根据不同上下文，可将消息头分为：

- General headers: 同时适用于请求和响应消息，但与最终消息主体中传输的数据无关的消息头。
- Request headers: 包含更多有关要获取的资源或客户端本身信息的消息头。
- Response headers: 包含有关响应的补充信息，如其位置或服务器本身（名称和版本等）的消息头。
- Entity headers: 包含有关实体主体的更多信息，比如主体长(Content-Length)度或其MIME类型。

<!-- markdownlint-disable MD013 MD033-->

| Header                                   | 类型                                  | 描述                                                         |
| ---------------------------------------- | ------------------------------------- | ------------------------------------------------------------ |
| `Authorization`                          | 字符串                                 | 请求的授权请求头                                               |
| `Date`                                   | Date                                  | 基于客户端时钟的请求日期和时间格式 [RFC 5322](https://tools.ietf.org/html/rfc5322#section-3.3) 。服务器不应对客户端时间做任何假设。提供此请求头时，必须将格林威治标准时间（GMT）用作该标头的时区参考。例如：`Wed, 24 Aug 2016 18:41:30 GMT`。请注意，GMT完全等于UTC（世界标准时间）。 |
| `Accept`                                 | 用户期望的 MIME 类型列表                 | 请求的响应内容类型，例如：<br/><ul><li> `application/xml` </li><li> `text/xml` </li><li> `application/json` </li><li> `text/javascript (for JSONP)` </li></ul><br/> 根据 HTTP 准则，这只是一个提示，响应可能具有不同的内容类型，例如 blob 提取，其中成功的响应将只是 blob 流作为有效负载。对于遵循 OData 的服务，应遵循 OData 中指定的优先顺序。 |
| `Accept-Encoding`                        | Gzip, deflate                         | 如果适用，REST 端点应该支持 GZIP 和 DEFLATE 编码。对于非常大的资源，服务可以忽略并返回未压缩的数据。 |
| `Accept-Language`                        | `en` ， `zh` ， 等。                      | 指定响应的首选语言。服务器可能不支持此功能，但是如果服务支持本地化，则必须通过 Accept-Language 请求头来实现。 |
| `Accept-Charset`                         | 字符集类型，例如 `UTF-8`               | 默认值为 `UTF-8` ，但是服务应该能够处理 `ISO-8859-1` 。              |
| `Content-Type`                           | 内容类型                              | 请求主体的 MIME 类型（PUT / POST / PATCH）                     |
| `Prefer`                                 | `return=minimal`，`return=representation` | 如果指定首选项 `return=minimal` ，则服务应响应成功的插入或更新而返回空主体。如果指定了`return=representation`，则服务应在响应中返回创建或更新的资源。如果客户端有时需要响应内容，但有时需要避免带宽太大的影响，则服务必须支持此请求头 |
| `If-Match`, `If-None-Match` , `If-Range` | 字符串                                | 支持使用乐观并发控制更新资源的服务必须支持 `If-Match 标头`。服务也可以使用其他与 ETag 相关的标头，只要它们遵循 HTTP 规范即可。 |

<!-- markdownlint-restore -->
### 6.4 响应格式

基于 Web 的通信，尤其是在涉及移动或其他低带宽客户端时，由于多种原因，已朝 JSON 方向快速发展，包括其轻量的趋势以及与基于 JavaScript 的客户端的易用性。

JSON 属性名称应为驼峰式。

服务应提供 JSON 作为默认编码，即请求头默认为 `Accept=application/json` 。

### 6.5 状态码

HTTP 响应状态代码指示特定 HTTP 请求是否已成功完成。响应分为五类：信息响应( `100` – `199` )，成功响应( `200` – `299` )，重定向( `300`–`399` )，
客户端错误( `400` – `499` )和服务器错误 ( `500` – `599` )。状态代码由 [section 10 of RFC 2616](https://tools.ietf.org/html/rfc2616#section-10) 定义。

RESTful 服务使用 HTTP 状态代码来指定 HTTP 方法执行的结果。 HTTP 协议使用整数和消息指定请求执行的结果。该数字称为**状态码**，该消息称为**原因短语**。
原因短语是人类可读的消息，用于阐明响应的结果。 HTTP 协议将范围内的状态代码分类。

#### 6.5.1 状态码范围

响应 API 请求时，必须使用以下状态代码范围。

| 范围   | 含义                                                                                         |
| ----- | ------------------------------------------------------------------------------------------- |
| `2xx` | 成功执行。方法执行有可能以多种方式成功执行。此状态代码指定成功的方式。                                  |
| `4xx` | 通常，这些问题包括请求，请求中的数据，无效的身份验证或授权等。在大多数情况下，客户端可以修改其请求并重新提交。 |
| `5xx` | 服务器错误：由于站点中断或软件缺陷，服务器无法执行该方法。5xx 范围状态代码不应用于验证或逻辑错误处理。       |

#### 6.5.2 允许的状态代码列表

- API 绝不能返回此表中未定义的状态代码。
- API 只能返回此表中定义的某些状态代码。

<!-- markdownlint-disable MD013-->
| 状态码                           | 描述                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| `200 OK`                        | 请求成功。                                           |
| `201 Created`                   | 用作对 `POST` 方法执行的响应，以指示成功创建资源。如果已经创建了资源（例如，通过先前执行同一方法），则服务器应返回状态代码 `200 OK` 。通常是在POST请求，或是某些PUT请求之后返回的响应。 |
| `202 Accepted`                  | 用于异步方法执行，以指定服务器已接受该请求并将在以后执行。有关更多详细信息，请参阅“[异步操作”](https://github.com/paypal/api-standards/blob/master/patterns.md#asynchronous-operations)。 |
| `204 No Content`                | 服务器成功处理了请求，但不需要返回任何实体内容，并且希望返回更新了的元信息。响应可能通过实体头部的形式，返回新的或更新后的元信息。如果存在这些头部信息，则应当与所请求的变量相呼应。如果客户端是浏览器的话，那么用户浏览器应保留发送了该请求的页面，而不产生任何文档视图上的变化，即使按照规范新的或更新后的元信息应当被应用到用户浏览器活动视图中的文档。由于 `204` 响应被禁止包含任何消息体，因此它始终以消息头后的第一个空行结尾。 |
| `400 Bad Request`               | 服务器无法理解该请求。使用此状态代码来指定：请求数据有误（请求的数据不能转换为基础数据类型、数据不是预期的数据格式、必填字段不可用、简单数据验证类型的错误）。 |
| `401 Unauthorized`              | 该请求需要身份验证，但未提供任何身份。请注意此与之间的区别 `403 Forbidden` 。 |
| `403 Forbidden`                 | 服务器已经理解请求，但是拒绝执行它。客户端无权访问资源，尽管它可能具有有效的凭据。如果业务级别授权失败， API 可以使用此代码。例如，持有者的资金不足。 |
| `404 Not Found`                 | 服务器未找到与请求 URI 匹配的任何内容。这意味着 URI 不正确或资源不可用。例如，可能是该键处的数据库中没有数据。 |
| `405 Method Not Allowed`        | 服务器尚未实现请求的 HTTP 方法。这通常是 API 框架的默认行为。    |
| `406 Not Acceptable`            | 当请求的资源的内容特性无法满足请求头中的条件，因而无法生成响应实体。例如，如果客户端发送了 `Accept: application/xml` 标头，而 API 只能返回 `application/json` ，则服务器必须返回 `406` 。 |
| `415 Unsupported Media Type`    | 当请求的有效负载的媒体类型无法处理时，服务器必须返回此状态代码。例如，如果客户端发送 `Content-Type: application/xml` 标头，但 API 仅接受 `application/json` ，则服务器必须返回`415`。 |
| `422 Unprocessable Entity`      | 请求的操作无法执行，可能需要与当前请求之外的 API 或进程进行交互。这与 500 响应不同，因为没有系统性的问题限制 API 执行请求。 |
| `429 Too Many Requests`         | 如果用户，应用程序或令牌的速率限制已超过预定义的值，则服务器必须返回此状态代码。在其他 HTTP 状态代码 [RFC 6585中](https://tools.ietf.org/html/rfc6585) 定义。 |
| `500 Internal Server Error`     | 这可能是系统错误，也可能是应用程序错误，通常表明尽管客户端似乎提供了正确的请求，但服务器上出现了意外情况。一个 `500` 响应指示服务器端软件缺陷或网站停止。 `500` 不应用于客户端验证或逻辑错误处理。 |
| `503 Service Unavailable`       | 由于临时维护，服务器无法处理服务请求。常见原因是服务器因维护或重载而停机。     |
<!-- markdownlint-restore -->

## 7. 命名规范

本节阐述在命名上的规范。

### 7.1 URI 规范

#### 7.1.1 URI 组成

URI遵循 [RFC 3986](https://tools.ietf.org/html/rfc3986) 规范。该规范简化了 REST API 服务的开发和使用。本节中的准则遵循 RFC 3986 约束来管理 URI 结构和语义。

根据 RFC 3986，通用 URI 语法由组件的层次结构序列组成，称为协议，权限，路径，查询字符串和锚点，如下面的示例所示。

```none
        https://example.com:8042/over/there?name=ferret#nose
        \___/   \_______________/\_________/\_________/\__/
          |           |            |            |        |
        scheme     authority      path        query   fragment
```

#### 7.1.2 URI 命名约定

以下是对API的 URI 特定命名约定准则的简要说明。本规范使用括号“()”进行分组，星号 “*” 指定零个或多个出现，括号“[]”表示可选字段。

- URI 必须以字母开头，并且只能使用小写字母。
- URI 路径中的文字或表达式应使用连字符（`-`）分隔。
- URI 路径和查询字符串必须将数据编码成 [UTF-8 八位字节](https://en.wikipedia.org/wiki/Percent-encoding)。
- URI 中应该使用名词复数来标识数据资源的集合。
  - `/invoices`
  - `/statements`
- 资源集合中的单个资源可以直接存在于集合 URI 的后面。
  - `/invoices/{invoice_id}`
- 子资源集合可以直接存在于单个资源之下。这应该传达与另一个资源集合的关系（在此示例中为发票项目）。
  - `/invoices/{invoice_id}/items`
- 子资源的单个资源可以存在，但应避免使用顶级资源。
  - `/invoices/{invoice_id}/items/{item_id}`
  - `/invoice-items/{invoice_item_id}`
- 资源标识符应遵循后续部分中描述的[建议](#7-2-资源名称)。

**例子：**

- `https://api.foo.com/v1/vault/credit-cards`
- `https://api.foo.com/v1/vault/credit-cards/CARD-7LT50814996943336KESEVWA`
- `https://api.foo.com/v1/payments/billing-agreements/I-V8SSE9WLJGY6/re-activate`

**正式定义：**

| 术语                    | 定义                                                         |
| ----------------------- | ------------------------------------------------------------ |
| URI                     | \[end-point\] '**/**' resource-path \['**?**'query'\]                      |
| end-point               | \[scheme "**://**" \]\[host \['**:**' port\]\]                 |
| schema                  | "**http**"或"**https** "                                  |
| 资源路径(resource-path)  | "**/v**" 版本 '**/**' 名称空间名称 '**/**' 资源（'**/**' 资源） |
| 资源(resource)          | 资源名称 ['**/**' 资源ID]                                    |
| 资源名称(resource-name)  | Alpha(Alpha \|数字\|'-')*                                  |
| 资源编号(resource-id)    | value                                                           |
| query                   | name '**=**' value ('**＆**' name=value）*                      |
| name                    | Alpha（Alpha \|数字\|'_'）*                                  |
| value                   | URI编码值                                              |

说明：

```none
'   用单引号括住特殊字符
"   用双引号引起来的字符串
()  使用括号进行分组
[]  使用方括号指定可选表达式
*   一个表达式可以重复零次或多次
```

### 7.2 资源名称

将服务建模为一组资源时，开发人员必须遵循以下原则：

- 必须使用名词，而不是动词。
- 单个资源名称必须为单数；集合的名称必须为复数。
  - 用户帐户上自动付款配置的说明。
    - `GET /autopay` 返回完整的表示形式。
  - 假设费用的集合：
    - `GET /charges` 返回已收取的费用清单。
    - `POST /charges` 创建一个新的费用资源， `/charges/1234` 。
    - `GET /charges/1234` 返回一次费用的完整表示。
- 资源名称必须为小写字母，并且只能使用字母数字字符和连字符。
- 连字符（-）必须用作 URI 路径文字中的单词分隔符。**请注意**，这是连字符用作单词分隔符的唯一位置。其他情况下，必须使用小驼峰式。

### 7.3 查询参数名称

- 查询字符串中的文字/表达式应使用小驼峰式。
- 查询参数值必须进行编码。
- 查询参数必须以字母开头，只能使用字母字符，数字。
- 查询参数应该是可选的。

### 7.4 字段名称

数据模型必须符合 [JSON](http://json.org/) 。这些值本身可以是对象，字符串，数字，布尔值或对象数组。

- 关键字名称必须为小写单词，并用小驼峰式。
  - `foo`
  - `barBaz`
- 前缀 `is` 或 `has` 的字段不应是布尔类型的 key。
- 代表数组的字段应使用复数名词来命名（例如， `authenticators` ：包含一个或多个 Authenticator， `products` ：包含一个或多个产品）。

## 8. 异常

根据 HTTP 规范，可以使用整数和消息指定请求执行的结果。该数字称为**状态码**，该消息称为**原因短语**。原因短语是一种人类可读的消息，用于阐明响应的结果。
 `4xx`  范围内的HTTP状态代码指示客户端错误（验证或逻辑错误），而  `5xx`  范围内的 HTTP 状态代码指示服务器端错误（通常是缺陷或中断）。
 但是，这些状态码和人类可读的原因短语不足以以机器可读的方式传达足够的错误信息。要解决错误，非人类的 RESTful API 使用者需要其他帮助。

异常字段：

```json
{
    "status": 400,
    "code": "some error code",
    "message": "error detail message",
    "timestamp": "2020-10-22T06:49:18.131+0000"
}
```

## 9. 超媒体

### 9.1 HATEOAS

超媒体是[超文本](https://en.wikipedia.org/wiki/Hypertext)一词的扩展，是一种非线性的信息介质，根据 [Wikipedia 的规定](https://en.wikipedia.org/wiki/Hypermedia)，
它包括图形，音频，视频，纯文本和超链接。超媒体作为应用程序状态的引擎 `HATEOAS` 是 Roy Fielding 在其[论文中](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)描述的 REST 应用程序体系结构的约束。

在 RESTful API 的上下文中，客户端可以完全通过服务动态提供的超媒体与服务交互。超媒体驱动的服务通过在响应中包括超媒体链接，向其客户端提供资源表示，
以动态导航 API 。这不同于 SOA 的其他形式，在 SOA 中，服务器和客户端基于 Web 上某个位置定义的或基于带外交换的，基于 WSD L的规范进行交互。

### 9.2 符合超媒体的 API

符合超媒体的 API 公开了服务的有限状态机。在这里，要求如 `DELETE` ， `PATCH` ， `POST` 和 `PUT` 通常在启动状态的转变，而响应指示状态的变化。
让我们以一个 API 为例，该 API 公开一组操作来管理用户帐户生命周期并实现HATEOAS接口约束。

客户端通过固定的URI开始与服务进行交互 `/users` 。此 URI 支持 `GET` 和 `POST` 操作。客户端执行一项 `POST` 操作以在系统中创建用户。

**要求**：

```none
POST https://api.foo.com/v1/customer/users

{
    "given_name": "James",
    "surname" : "Greenwood",
    ...

}
```

**响应：**

API 从输入中创建一个新用户，并在响应中将以下链接返回到客户端。

- 检索用户完整表示的 `self` 链接（`GET`）。
- 更新用户的链接（`PUT`）。
- 部分更新用户的链接（`PATCH`）。
- 删除用户的链接（`DELETE`）。

```none
{
HTTP/1.1 201 CREATED
Content-Type: application/json
...

"links": [
    {
        "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
        "rel": "self",

    },
    {
        "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
        "rel": "delete",
        "method": "DELETE"
    },
    {
        "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
        "rel": "replace",
        "method": "PUT"
    },
    {
        "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
        "rel": "edit",
        "method": "PATCH"
    }
]

}
```

客户端可以将这些链接存储在其数据库中以供以后使用。

然后，在管理员决定删除其中一个用户之前，客户端可能希望显示一组用户及其详细信息。因此，客户端对相同的 URI `/users` 执行 `GET` 操作。

**请求**：

```none
GET https://api.foo.com/v1/customer/users
```

该API通过相应的 `self` 链接返回系统中的所有用户。

**响应：**

```none
{
    "total_items": "166",
    "total_pages": "83",
    "users": [
    {
        "given_name": "James",
        "surname": "Greenwood",
        ...
        "links": [
            {
                "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
                "rel": "self"
            }
        ]
    },
    {
        "given_name": "David",
        "surname": "Brown",
        ...
        "links": [
            {
                "href": "https://api.foo.com/v1/customer/users/ALT-MDFSKFGIFJ86DSF",
                "rel": "self"
            }
   },
   ...
}
```

客户端可以跟随 `self` 用户的链接并找出它可以在用户资源上执行的所有可能的操作。

**请求：**

```none
GET https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI
```

**响应：**

```none
HTTP/1.1 200 OK
Content-Type: application/json
{
        "given_name": "James",
        "surname": "Greenwood",
        ...
        "links": [
        {
            "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
            "rel": "self",

        },
        {
            "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
            "rel": "delete",
            "method": "DELETE"
        },
        {
            "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
            "rel": "replace",
            "method": "PUT"
        },
        {
            "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
            "rel": "edit",
            "method": "PATCH"
        }


}
```

为了删除用户，客户端从其数据存储中检索 `delete` 链接的 URI 并对该 URI 执行删除操作。

**请求：**

```none
DELETE https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI
```

综上所述：

- 客户端可以导航到 API 的入口点，以便访问所有其他资源。
- 客户端无需构建组成 URI 的逻辑来执行不同的请求或通过查看可能与 URI 和状态更改相关的响应详细信息（在后面的部分中进行更详细的描述）来编码任何种类的业务规则。 。
- 客户端承认创建 URI 的过程属于服务器这一事实。
- 客户端将 URI 视为不透明标识符。
- 在表示中使用超媒体的 API 可以无缝扩展。随着新方法的引入，可以通过相关的 HATEOAS 链接扩展响应。这样，客户可以增量方式利用该功能。例如，如果 API 开始支持新 `PATCH` 操作，则客户端可以使用它进行部分更新。

仅仅是链接的存在并不能使客户端从需要了解数据来请求转换和所有相关的链接语义中解脱出来，特别是对 `POST` 、 `PUT` 、 `PATCH` 操作。 API 必须提供文档，以清楚地描述每个 URI 的所有链接，链接关系类型和请求响应格式。

后续部分提供有关链接结构以及不同关系类型的含义的更多详细信息。

#### 9.2.1 链接描述对象

必须使用[链接描述对象（LDO）](http://json-schema.org/latest/json-schema-hypermedia.html#anchor17)模式描述链接。LDO 在 links 数组中描述了单个链接关系。以下是链接描述对象的属性的简要描述。

- `href`：
  - `href`必须提供属性的值。
  - `href`属性的值必须是 [URI 模板](https://tools.ietf.org/html/rfc6570)，用于确定相关资源的目标 URI。应根据 [RFC 6570](https://tools.ietf.org/html/rfc6570) 将其解析为 URI 模板。
  - 仅将绝对 URI 用作`href`属性值。客户端通常将表示形式的链接关系类型的绝对 URI 标记为书签，以便以后发出 API 请求。
    开发人员必须使用 [URI 组件命名约定](https://github.com/paypal/api-standards/blob/master/api-style-guide.md#uri-component-names)来构造绝对 URI 。
    传入 `Host` 请求头（例如 api.foo.com ）中的值必须用作 `host` 绝对 URI 的字段。
- `rel`：
  - `rel` 代表“链接关系类型”中定义的关系
  - 该`rel`属性的值指示与目标资源的关系的名称。
  - `rel`必须提供属性的值。
- `method`：
  - 该 `method` 属性标识必须用于向链接目标发出请求的 HTTP 动词。如果忽略该 `method` 属性，则默认值为 `GET` 。
- `title`：
  - 该 `title` 属性提供了链接的标题，并且是一个有用的文档工具，可帮助最终客户理解。此属性不是必要的。

##### 不为 LDO 使用 HTTP 请求头

请注意，这些 API 准则不建议使用 HTTP `Location` 标头来提供链接。另外，他们也不建议使用 [JAX-RS 中](https://java.net/projects/jax-rs-spec/pages/Hypermedia) `Link` 描述的标头。HTTP 请求头的范围仅限于客户端和服务之间的点对点交互。由于响应可能被传递到客户端上的其他层和组件，这些层和组件可能与服务不直接交互，因此请求头中存储的任何信息都可能不可用。

#### 9.2.1 链接数组

`links` 模式的 array 属性用于将链接描述对象与 [JSON hyper-schema draft-04] [3](http://json-schema.org/draft-04/hyper-schema#) 实例相关联。

- 此属性必须是一个数组。
- 数组中的项目必须为链接描述对象类型。

##### 指定链接数组

这是一个如何描述架构中的链接的示例。

- 必须在 API 资源模式定义的一部分中提供与下面的示例 JSON 模式中定义的链接数组相似的链接数组。请注意， links 数组需要在 `properties` 对象的关键字内声明。这是代码生成器为所生成的对象中的 links 数组添加 setter 、 getter 方法所必需的。
- 必须使用 URI 模板在响应模式中声明 API 作为响应的一部分返回的所有可能的链接。 URI 模板的 links 数组必须在 `properties` 关键字之外声明。

```none
{
    "type": "object",
    "$schema": "http://json-schema.org/draft-04/hyper-schema#",
    "description": "A sample resource representing a customer name.",
    "properties": {
        "id": {
        "type": "string",
            "description": "Unique ID to identify a customer."
        },
        "first_name": {
            "type": "string",
            "description": "Customer's first name."
        },
        "last_name": {
            "type": "string",
            "description": "Customer's last name."
        },
        "links": {
            "type": "array",
            "items": {
                "$ref": "http://json-schema.org/draft-04/hyper-schema#definitions/linkDescription"
            }
        }
    },
    "links": [
        {
            "href": "https://api.foo.com/v1/customer/users/{id}",
            "rel": "self"
        },
        {
            "href": "https://api.foo.com/v1/customer/users/{id}",
            "rel": "delete",
            "method": "DELETE"
        },
        {
            "href": "https://api.foo.com/v1/customer/users/{id}",
            "rel": "replace",
            "method": "PUT"
        },
        {
            "href": "https://api.foo.com/v1/customer/users/{id}",
            "rel": "edit",
            "method": "PATCH"
        }
    ]

}
```

以下是符合上述架构的示例响应。

```none
{
    "id": "ALT-JFWXHGUV7VI",
    "first_name": "John",
    "last_name": "Doe",
    "links": [
        {
            "href": "https://api.foo.com/v1/cusommer/users/ALT-JFWXHGUV7VI",
            "rel": "self"
        },
        {
            "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
            "rel": "edit",
            "method": "PATCH"
        }
    ]
}
```

### 9.3 链接关系类型

`Link Relation Type` 用作链接的标识符。 API 必须分配有意义的链接关系类型，该类型必须明确描述链接的语义。客户端使用相关的链接关系类型，以便从表示形式中识别要使用的链接。

当 [IANA 的标准链接关系列表] [5](http://www.iana.org/assignments/link-relations/link-relations.xhtml) 中定义的链接关系类型的语义与您要定义的语义匹配时，则必须使用它。下表描述了一些常用的链接关系类型。它还列出了这些准则定义的一些其他lin关系类型。

| 链接关系类型     | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| `self`           | 传达链接上下文的标识符。通常是指向资源本身的链接。           |
| `create`         | 指可用于创建新资源的链接。                                   |
| `edit`           | 指编辑（或部分更新）链接标识的表示形式。使用它来表示`PATCH`操作链接。 |
| `delete`         | 指删除链接标识的资源。使用它`Extended link relation type`来表示`DELETE`操作链接。 |
| `replace`        | 指完全更新（或替换）链接标识的表示形式。使用它`Extended link relation type`来表示`PUT`操作链接。 |
| `first`          | 指结果列表的第一页。                                         |
| `last`           | 指提供的结果列表的最后一页`total_required`指定为查询参数。   |
| `next`           | 指结果列表的下一页。                                         |
| `prev`           | 指结果列表的前一页。                                         |
| `collection`     | 引用集合资源（例如/ v1 / users）。                           |
| `latest-version` | 指向包含最新（例如当前）版本的资源。                         |
| `search`         | 指可用于搜索链接上下文和相关资源的资源。                     |
| `up`             | 指资源层次结构中的父资源。                                   |

对于所有 [`controller`](https://github.com/paypal/api-standards/blob/master/patterns.md#controller-resource) 款式复杂的操作，控制器 `action` 名称必须使用中的链接关系类型（例如`activate`，`cancel`，`refund`）。

## 10. CORS

跨源资源共享标准新增了一组 HTTP 头，允许服务器声明哪些源站通过浏览器有权限访问哪些资源。另外，规范要求，对那些可能对服务器数据产生副作用的 HTTP 请求方法（特别是 GET 以外的 HTTP 请求，
或者搭配某些 MIME 类型的 POST 请求），浏览器必须首先使用 OPTIONS 方法发起一个预检请求（preflight request），从而获知服务端是否允许该跨源请求。
服务器确认允许之后，才发起实际的 HTTP 请求。在预检请求的返回中，服务器端也可以通知客户端，是否需要携带身份凭证（包括 Cookies 和 HTTP 认证相关数据）。

符合 REST API 准则的服务必须支持 [CORS（跨源资源共享）](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)。

服务应支持信任的 CORS 来源，并通过有效的 OAuth 令牌强制执行授权。服务不应通过来源验证支持用户凭据。特殊情况可能会有例外。

### 10.1 指导说明

Web 开发人员通常不需要做任何特殊的事情就可以利用 CORS 。所有步骤都应该有框架或者库自动执行。

#### 10.1.1 避免预检

由于 CORS 协议可以触发预检请求，这些预检请求会增加与服务器的交互，因此对性能要求较高的应用可能会希望避免这种情况。CORS 可以对简单的请求不要求预检，但对于其他请求一定要预检。

**简单请求：**

必须满足如下所有条件的请求才能定义为简单请求

- 使用下列方法之一
  - `GET`
  - `HEAD`
  - `POST`
- 除了被用户代理自动设置的标头字段（例如 Connection ，User-Agent）和在 Fetch 规范中定义为禁用标头名称外的其他标头，允许人为设置的字段为 Fetch 规范定义的对 CORS 安全的首部字段集合。该集合为：
  - `Accept`
  - `Accept-Language`
  - `Content-Language`
  - [`Content-Type`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type) （需要注意额外的限制）
  - `DPR`
  - `Downlink`
  - `Save-Data`
  - `Viewport-Width`
  - `Width`
- `Content-Type` 的值仅限于下列三者之一：
  - `text/plain`
  - `multipart/form-data`
  - `application/x-www-form-urlencoded`

### 10.2 服务指南

- 了解浏览器在跨域请求中发送的 `Origin` 请求头，以及在检查访问的预检 `OPTIONS` 请求中发送的 `Access-Control-Request-Method` 请求标头。
- 如果请求中存在 `Origin` 头：
  - 如果请求使用 `OPTIONS` 方法并包含 `Access-Control-Request-Method` 标头，则它是预检请求，旨在在实际请求之前探测访问。否则，这是一个正常的请求。对于预检请求，除了执行以下添加标头的步骤之外，服务务必不要执行任何其他处理，并且必须返回 `200 OK` 。对于非预检请求，除了请求的常规处理之外，还会添加以下头。
  - 将 `Access-Control-Allow-Origin` 头添加到响应中，该头包含与 `Origin` 头相同的值。请注意，这需要服务动态生成请求头值。不需要 `cookie` 或任何其他形式的[用户凭证的资源](http://www.w3.org/TR/access-control/#user-credentials)可以用通配符星号（*）代替。请注意，通配符仅在此处可接受，不适用于以下所述的任何其他头。
  - 如果调用方需要访问不在[简单响应标头](http://www.w3.org/TR/access-control/#simple-hader)列表中的响应标头（ `Cache-Control` ， `Content-Language` ， `Content-Type` ， `Expires` ， `Last-Modified` ， `Pragma` ），
    则应将客户端有权访问的其他响应头名称添加到 `Access-Control-Expose-Headers` 标头中。
  - 如果请求需要 cookie ，则添加一个 `Access-Control-Allow-Credentials` 标头设置为 `true`。
  - 如果该请求是预检请求（请参阅第一个项目符号），则该服务必须：
    - 添加一个 `Access-Control-Allow-Headers` 响应标头，其中包含允许客户端使用的请求标头名称的列表。该列表仅需要包含不在[简单请求标头](http://www.w3.org/TR/access-control/#simple-header)集中的
      [标头](http://www.w3.org/TR/access-control/#simple-header)（ `Accept` ， `Accept-Language` ， `Content-Language` ）。如果服务接受的头没有任何限制，则该服务可以简单地返回与客户端发送的 `Access-Control-Request-Headers` 头相同的值。
    - 添加一个 `Access-Control-Allow-Methods` 响应标头，其中包含允许调用方使用的 HTTP 方法的列表。

添加一个 `Access-Control-Max-Age` 响应标头，其中包含该响应有效的时间（因此可以在随后的实际请求之前避免使用）。请注意，虽然习惯使用较大的值，例如 2592000（30天），但许多浏览器会自行施加一个较低的限制（例如5分钟）。

由于浏览器的预检响应缓存非常差，因此预检响应带来的额外请求次数会影响性能。性能至关重要的交互式 Web 客户端使用的服务时应避免导致预检请求的模式

- 对于 GET 和 HEAD 调用，请避免要求不属于上述简单请求标头。允许将它们作为查询参数提供。
  - `Authorization` 标头不是简单请求头的一部分，因此对于需要身份验证的资源，必须通过 `access_token` 查询参数发送身份验证令牌。请注意，不建议在 URL 中传递身份验证令牌，因为它可能导致令牌记录在服务器日志中并暴露给有权访问这些日志的任何人。通过URL接受身份验证令牌的服务必须采取措施来减轻安全风险，例如使用短暂的身份验证令牌，禁止身份验证令牌被记录以及控制对服务器日志的访问。
- 避免要求 Cookie 。如果设置了 `withCredentials` 属性，则 XmlHttpRequest 仅在跨域请求上发送 cookie 。这也会导致预检请求。
  - ~~需要基于 Cookie 的身份验证的服务务必使用 “Dynamic Canary” 来保护所有接受 Cookie 的 API 。~~
- 对于 POST 调用，请在适用的情况下使用简单的 `Content-Type` （`aplication/x-www-form-urlencoded` ，`multipart/form-data` ， `text/plain` ）。任何其他 `Content-Type` 都会引发预检请求。
  - 服务不得以避免 CORS 预检请求的名义违反其他 API 建议。特别是，根据建议，使用 `Content-Type` 的大多数 POST 请求实际上都需要进行预检请求。
  - 如果必须取消预检，那么服务应支持替代的数据传输机制，但是必须也支持推荐的方法。

另外，在某些时候，可以支持 JSONP 模式，以进行简单的，仅限 GET 的跨域访问。在 JSONP 中，服务采用指示格式（`$format=json`）的参数和指示回调（`$callback=someFunc`）的参数，并返回 `text/ javascript` 文档，该文档包含包装在具有指定名称的函数调用中的 JSON 响应。 Wikipedia 上有关 JSONP 的更多信息：[JSONP](https://en.wikipedia.org/wiki/JSONP)。

## 11. JSON

### 11.1  JSON 类型

本节提供了与 JSON 基本类型的用法有关的准则，以及地址，名称，货币，货币，国家，电话等常用的 JSON 类型。

#### 11.1.1 JSON 基本类型

JSON Schema [draft-04](http://tools.ietf.org/html/draft-zyp-json-schema-04)应该用于定义 API 中的所有字段。因此，应注意以下有关 JSON Schema 原语类型的注释。以下是指导使用 JSON 基本类型表示形式的准则。

##### 11.1.1.1 字符串

`strings` 应始终明确定义 `minLength` 和 `maxLength` 。

这样做有几个原因。

1. 没有最大长度，就不可能可靠地定义数据库列来存储给定的字符串。
2. 没有最大值和最小值，也无法预测长度的变化是否会破坏与现有客户端的向后兼容性。
3. 最后，没有最小长度，客户端通常有可能在不允许发送空字符串的情况下发送空字符串。

API 可能避免定义 `minLength` 和 `maxLength` 仅如果字符串值是从已经拒绝提供这些值的任何信息的另一上游系统。该决定必须记录在架构中。

API 的作者在定义时应考虑实际限制 `maxLength` 。例如，当使用 VARCHAR2 类型时，现代版本的 Oracle 可以安全地存储不超过 1,000 个字符的 Unicode 字符串。（大小限制为[4,000](https://docs.oracle.com/cd/B28359_01/server.111/b28320/limits001.htm)个字节，每个 Unicode 字符最多可能需要四个字节来存储）。

`string` 应该`pattern`适当地利用该属性，尤其是在定义枚举值或数字时。但是，建议不要在没有充分技术理由的情况下过度约束字段。

##### 11.1.1.2 枚举

JSON Schema `enum` 关键字很难安全使用。`enum` 在不破坏向后兼容性的情况下，无法在描述服务响应的架构中向中添加新值。在这种情况下，客户端通常会拒绝其值不在所提出的模式的较早副本中的响应。这通常不是所需的行为。客户端通常应该更轻松地处理未知值，但是由于您无法控制或验证其行为，因此添加新的枚举值并不安全。

由于上述原因，架构作者在使用 `enum` JSON类型的时必须遵守以下准则 `string` 。

- 关键字 `enum` 应该仅在固定的一组值时使用，以后再也不会更改。
- 如果您预计 `enum` 将来会向数组添加新值，请避免使用关键字 `enum` 。您应该改用一种 `string` 类型，并为记录所有可接受的值 `string` 。当使用 `string` 类型表示枚举值时，应通过字段强制执行[命名约定](https://github.com/paypal/api-standards/blob/master/api-style-guide.md#enum-names)`pattern`。
- 如果没有技术上的理由（例如，一个较小的现有数据库列） `maxLength` 应设置为 255 。`minLength` 应设置为1以防止客户端发送空字符串。
- `enum` 在文档中应精确定义字段的所有可能值。如果 `description` 字段中没有足够的空间来执行此操作，则应使用API的用户指南进行定义。

下面给出的是用于强制命名约定和长度约束的JSON代码段。

##### 11.1.1.3 数值

`number`JSON中的类型有很多问题。

JSON 本身定义了一个非常简单的值 `number` ：它是一个无界的定点值。这是由铁路图所示以及用于`number`在[JSON](http://json.org/)。`number`JSON中只有一种类型；没有单独的`integer`类型。

JSON模式与JSON不同，并定义了两种数字类型： `number`和`integer` 。纯粹是为了方便模式验证。 JSON 数字类型用于实现两者。就像 JSON 中一样，除非模式作者提供显式 `minimum` 和 `maximum` 值，否则这两种类型都是不受限制的。

许多编程语言不能安全地处理 JSON 中的无界值。 JavaScript 是一个很好的例子。JSON 解析器是 [ECMAScript](http://www.ecma-international.org/publications/standards/Ecma-262.htm) 规范的一部分。
但是，它要求将所有 JSON 数字反序列化为JavaScript 支持的唯一类型 -64 位浮点数 `number`。这意味着在 JavaScript 客户端中尝试反序列化大于约 2^53 的任何 JSON 数字将导致异常。

为了确保最大程度的跨客户端兼容性，模式作者应：

- 切勿使用JSON模式 `number` 类型。某些语言可能会将其解释为定点值，而某些语言可能会将其解释为浮点数。始终用于 `string` 表示十进制值。
- 仅定义 `integer` 可以用32位带符号整数表示的值的类型，即（ `(2^31) - 1) and -(2^31` ）之间的值。这样可以确保在多种编程语言和环境中的兼容性。例如，JavaScript中的数组索引是带符号的32位整数。
- 使用`integer`类型时，请始终提供明确的最小值和最大值。这不仅可以检测到向后不兼容的更改，还可以确保所有值都可以容纳在32位带符号整数中。如果没有其他技术原因，则将 `maximum` 和 `minimum` 分别定义为 `2147483647 (((2^31) - 1))` 和 `-2147483648 (-(2^31))` 或 `0` 。应该使用常识来确定是否允许使用负值。通常不应使用将来可能发生变化的业务逻辑来确定界限；业务规则可以轻松更改并破坏向后兼容性。

如果现在或将来有可能无法用带符号的 32 位整数表示该值，请不要使用 JSON 模式 `integer` 类型。请 `string` 改用。

**例子**:

此 `integer` 类型可能用于定义数组索引或页数，或者可能定义了开设帐户的月数。

```json
{
    "type": "integer",
    "minimum": 0,
    "maximum": 2147483647
}
```

当使用 `string` 类型表示数字时，作者必须使用 `minLength` 和提供字号边界 `maxLength`，并限制字符串的定义以仅使用数字表示数字 `pattern` 。

例如，此定义仅允许正整数和零，最大值为999999：

```json
{
    "type": "string",
    "pattern": "^[0-9]+$",
    "minLength": 1,
    "maxLength": 6
}
```

以下定义允许以正数或负数表示定点十进制值，最大长度为32，并且对比例没有要求：

```json
{
    "type": "string",
    "pattern": "^(-?[0-9]+|-?([0-9]+)?[.][0-9]+)$"
    "maxLength": 32,
    "minLength": 1,
}
```

##### 11.1.1.4 数组

JSON 定义 `array` 为无界。

尽管由于内存限制，实际的限制通常要低得多，但是许多编程语言的确对数组的大小设置了最大的理论限制。例如， ECMA-262 规范将 JavaScript 限制为 32位 无符号整数的长度。 Java 被[限制为 `Integer.MAX_VALUE - 8`](https://stackoverflow.com/questions/3038392/do-java-arrays-have-a-maximum-size#3039805)。

为了确保跨语言的最大兼容性并鼓励使用分页的 API ，`maxItems` 应始终由架构作者定义。 `maxItems` 不应具有大于 16 位有符号整数（ `32767` 即或）可以表示的值 `(2^15) - 1)` 。

注意，开发人员可以选择设置较小的值。 当没有更好的选择可用时， `32767` 该值是默认选择。但是，开发人员应该设计 API 时应兼顾未来数据的增长。例如，尽管现在分页的 API 最多每页最多只能支持 100 个结果，但是性能改进可以使开发人员在明年将其提高到 1,000 个结果。因此， `maxItems` 不应用于是最大页面大小。

`minItems` 可以定义。在大多数情况下，其值为`0`或`1`。

##### 11.1.1.5 空值

API 不得生成或使用 `null` 值。

`null` 是JSON中的原始类型。当根据 JSON Schema 验证 JSON 文档时，只有在模式明确允许的情况下，使用 `type` 关键字(例如 `{"type": "null"}` )，属性的值才能为 `null` 。
由于在 API  中， `type` 将始终用来定义其他值，例如 `object` 或 `string` ，并且这些标准禁止使用架构组合关键字（例如 `anyOf` 或 `oneOf` ，它允许多种类型），
如果 API 产生或出现空值，则根据 API 本身的架构，这实际上不太可能是有效数据。

另外，在 JSON 中，对象不应有被定义为 `undefined` 的属性；这在概念上与用 `null` 值定义的属性是分离的，但是许多编程语言都难以区分这种区别。

例如，被定义为 `{"type": "null"}` 的属性 `myProperty` 可以表示：

```json
{
    "myProperty": null
}
```

而如果属性被定义为 `undefined` 则不会出现在对象中：

```json
{ }
```

在大多数强类型化语言（例如Java）中，没有 `undefined` 类型的概念，这意味着对于JSON对象中所有未定义的字段，反序列化器将返回此类类型的值，如 `null` 您尝试检索它们的值。
同样， `null` 即使基于序列化程序无法确定用于该属性的 Java 对象的作者是使用值定义 `null` 还是简单地定义，某些基于 Java 的 JSON 序列化程序默认将字段序列化为 JSON 。
例如，在 Jackson 中，通过使用[`JsonInclude`](https://fasterxml.github.io/jackson-annotations/javadoc/2.8/com/fasterxml/jackson/annotation/JsonInclude.html)注释来缓和这种行为。
另一方面， org.json 库定义了一个名为 [NULL](https://stleary.github.io/JSON-java/org/json/JSONObject.html#NULL) 的对象以区分 `null` 和 `undefined` 。

完全避免使用 JSON null 可以帮助避免许多这些细微的跨语言兼容性陷阱。

##### 11.1.1.6 其他

在模式对象中设置 [`additionalProperties`](http://json-schema.org/latest/json-schema-validation.html#anchor134)为 `false` 会破坏那些使用 API 的 JSON 模式（由其合同定义）
来验证 API请求和响应的客户端的向后兼容性。出于相同的原因，模式创建者务必不得将明确设置 `additionalProperties` 为 `false` 。

相反， API 实现应通过在运行时根据已定义的API合同对请求和响应进行严格验证来强制要求和对API合同的响应。

#### 11.1.2 常见类型

API中的资源表示必须在可能的情况下重用[公共数据类型](https://github.com/paypal/api-standards/blob/master/v1/schema/json/draft-04)定义。以下各节提供了有关其中一些常见类型的一些详细信息。有关更多详细信息，请参考[`README`](https://github.com/paypal/api-standards/blob/master/v1/schema/json/draft-04/README.md)和[模式](https://github.com/paypal/api-standards/blob/master/v1/schema/json/draft-04)。

##### 11.1.2.1 日期，时间和时区

处理日期和时间时，所有 API 都必须遵守以下准则。

- 日期和时间字符串必须符合 [RFC3339](https://www.ietf.org/rfc/rfc3339.txt) `5.6` 节中定义 `date-time` 的格式。在用例中，您仅需要 RFC3339 格式的字段的子集（例如 `full-date` 或 `full-time` ）。
- 所有 API 必须在响应只使用 [UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time) 时间（也叫 [名祖鲁时间](https://en.wikipedia.org/wiki/List_of_military_time_zones)
   或 [GMT](https://en.wikipedia.org/wiki/Greenwich_Mean_Time) ）。
- 处理请求时， API 应该接受 `date-time` 或包含 UTC 偏移量的时间字段。例如， `2016-09-28T18:30:41.000+05:00` 应该接受与等效 `2016-09-28T13:30:41.000Z` 。
  这有助于确保与在发送请求之前可能无法将值归一化为 UTC 的第三方的兼容性。在这种情况下，偏移量应仅用于计算等效的 UTC 时间，然后再将其保留在系统中（由于已知的平台/语言/数据库互操作性问题）。 UTC 偏移不得衍生其他任何内容。
- 如果业务逻辑要求表达事件的时区，建议您使用单独的请求/响应字段来显式获取时区。您不应该使用 offset 来获取时区信息。仅靠偏移量不足以将存储的 UTC 时间准确地转换回以后的本地时间。
  原因是许多地区的 UTC 偏移量可能相同，并且根据一年中的时间，可能还有其他因素，例如夏令时。例如，偏移量 `UTC-05:00` 代表冬季的东部标准时间，夏季的中部夏令时间，以及巴拿马，哥伦比亚和秘鲁的全年偏移量。
- 时区字符串务必是每个 [IANA 时区数据库](https://www.iana.org/time-zones)（又名 **Olson** 数据库， **tzdata** 或 **zoneinfo** 数据库），例如，太平洋时间的 `America/ Los_Angeles` ，中欧的 `Europe/Berlin` 时间。
- 当在请求或响应中表达与特定时区（例如用户的出生日期，到期日期，发布日期等）无关的[浮点时间](https://www.w3.org/International/wiki/FloatingTime)值时，
   API 不应将其与时区相关联。原因是 UTC 偏移会更改浮动时间值的含义。例如，所有时区在本初子午线以西的国家都将浮点时间值视为前一天。

## 12. 版本化

API 的版本控制必须遵循一下原则：

- API 规范必须遵循版本控制方案，在版本号 `v1.2` 中，第一个数字是主版本号，第二个数字是次版本号
- 每当对 API 进行增量更改时，无论是否向后兼容，都必须对 API 规范进行版本控制。这允许对变更进行标记、记录、评审、讨论、发布和交流。
- API 地址只能反映主要版本。
- API 版本反映了接口更改，可以与服务版本控制方案分开。
- 次要 API 版本必须保持与同一主要版本中所有以前的次要版本的向后兼容性。
- 主要的API版本可以保持与先前主要版本的向后兼容性。

### 12.1 向后兼容

API 应该以向前和可扩展的方式进行设计，以保持兼容性并避免资源，功能和过多版本控制的重复。

API必须遵循以下原则才能被认为是向后兼容的：

- 所有修改都必须包含
- 所有修改都必须是可选的
- 语义一定不能改变
- 查询参数和请求主题参数必须无序
- 必须为现有资源实现其他功能
  - 作为可选扩展，或
  - 作为对新子资源的操作，或
  - 如果不能合理扩展现有操作（例如：创建资源），则通过更改请求主题，同时仍然可以接受先前所有存在的请求变化

## 13. 异步处理

### 13.1 上下文问题

在现代应用程序开发中，客户端应用程序（通常在 Web 客户端（浏览器）中运行的代码）依赖远程 API 来提供业务逻辑和组合功能是很正常的。
这些API可能与应用程序直接相关，也可能是第三方提供的共享服务。通常，这些API调用通过HTTP（S）协议进行，并遵循REST语义。

在大多数情况下，客户端应用程序的API被设计为可快速响应，数量级为100毫秒或更短。许多因素都会影响响应延迟，包括：

- 应用程序的托管堆栈。
- 安全组件。
- 调用者和后端的相对地理位置。
- 网络基础设施。
- 当前负载。
- 请求有效负载的大小。
- 处理队列长度。
- 后端处理请求的时间。

这些因素中的任何一个都会增加响应的延迟。通过扩展后端可以缓解某些问题。其他诸如网络基础结构之类的应用程序在很大程度上不受应用程序开发人员的控制。
大多数API可以足够迅速地做出响应，以使响应可以通过同一连接返回。应用程序代码可以以非阻塞方式进行同步API调用，从而呈现出异步处理的外观，建议对I / O绑定操作进行异步处理。

但是，在某些情况下，后端完成的工作可能会耗时数秒，或者可能是在几分钟甚至几小时内执行的后台进程。
在这种情况下，在响应请求之前等待工作完成是不可行的。对于任何同步请求-应答模式，这种情况都是潜在的问题。

一些体系结构通过使用消息代理来分隔请求和响应阶段来解决此问题。通常通过使用[基于队列的负载均衡模式](https://docs.microsoft.com/en-us/azure/architecture/patterns/queue-based-load-leveling)来实现这种分离。
这种分离可以允许客户端进程和后端API独立扩展。但是，当客户端需要成功通知时，这种分离也会带来额外的复杂性，因为此步骤需要变得异步。

针对客户端应用程序讨论的许多相同考虑因素，也适用于分布式系统（例如，微服务体系结构）中的服务器到服务器REST API调用。

### 13.2 解决方法

解决此问题的一种方法是使用HTTP轮询。轮询对客户端代码很有用，因为很难提供回调端点或使用长时间运行的连接。
即使可能进行回调，有时所需的额外库和服务也会增加过多的额外复杂性。

- 客户端应用程序对API进行同步调用，从而在后端触发长时间运行的操作。
- API尽可能快地同步响应。它返回HTTP 202（已接受）状态码，确认已接收到请求进行处理。

- 该响应包含指向客户端可以轮询以检查长期运行结果的端点的位置引用。
- API将处理工作转移到另一个组件，例如消息队列。
- 在工作仍处于待处理状态时，状态端点将返回HTTP202。完成工作后，状态端点可以返回指示完成的资源，或重定向到另一个资源URL。
  例如，如果异步操作创建了新资源，则状态端点将重定向到该资源的URL。

![异步HTTP请求的请求和响应流](https://docs.microsoft.com/en-us/azure/architecture/patterns/_images/async-request.png)

1. 客户端发送请求并接收HTTP 202（已接受）响应。
2. 客户端将HTTP GET请求发送到状态端点。工作仍在进行中，因此此调用还返回HTTP 202。
3. 在某个时候，工作完成，状态端点返回302（已找到），重定向到资源。
4. 客户端通过指定的URL获取资源。

您应该公开一个返回异步请求状态的终结点，以便客户端可以通过轮询状态终结点来监视状态。在202响应的Location标头中包含状态终结点的URI。例如：

```none
HTTP/1.1 202 Accepted
Location: /api/status/12345
```

如果客户端向该端点发送GET请求，则响应应包含请求的当前状态。可选地，它还可以包括估计的完成时间或取消操作的链接。

```none
HTTP/1.1 200 OK
Content-Type: application/json

{
    "status":"In progress",
    "link": { "rel":"cancel", "method":"delete", "href":"/api/status/12345" }
}
```

如果异步操作创建了新资源，则在操作完成后，状态端点应返回状态代码303（请参阅其他）。在303响应中，包含一个Location标头，该标头给出了新资源的URI：

```none
HTTP/1.1 303 See Other
Location: /api/orders/12345
```

## 14. Webhooks

### 14.1 范围

服务可以通过网络挂钩实现推送通知。本节介绍以下关键方案：

> 通过HTTP回调（通常称为Web挂钩）将通知推送到可公开寻址的服务器。

选择该方法的原因是它的简单性，广泛的适用性以及服务订户进入的门槛低。它仅作为最低要求，也是其他功能的起点。

### 14.2原则

支持Web挂钩的服务的核心原则是：

1. 服务必须至少实现戳/拉模型。在戳/拉模型中，将通知发送给客户端，然后客户端发送请求以获取当前状态或自上次通知以来的更改记录。
   这种方法避免了消息排序，遗漏消息和变更集的复杂性。服务可以添加更多数据以提供丰富的通知。
2. 服务必须实现用于配置回调URL的质询/响应协议。
3. 服务应有一个建议的过期期限，并应根据情况灵活调整服务。
4. 服务应该允许发出成功通知的订阅永久存在，并且应该容忍合理的中断时间。
5. Firehose订阅必须仅通过HTTPS交付。服务应该要求其他订阅类型为HTTPS。有关更多详细信息，请参见“安全性”部分。

### 14.3订阅类型

订阅有两种类型，服务可以既实现又不实现。支持的订阅类型为：

1. Firehose订阅–通常在应用程序注册门户中为订阅应用程序手动创建订阅。任何用户已同意接收应用程序的活动通知将发送到此单个订阅。
2. 每个资源的订阅–订阅应用程序使用代码在运行时以编程方式为某些特定于用户的实体创建订阅。

同时支持两种订阅类型的服务应为两种类型提供差异化的开发人员体验：

1. Firehose –服务必须不要求开发人员创建代码，除非直接验证并响应通知。服务必须提供用于订阅管理的管理UI。服务不应假设最终用户知道订阅，而只能假设订阅应用程序的功能。
2. 每用户–服务必须为开发人员提供一个API，以作为其应用程序的一部分来创建和管理订阅以及验证和响应通知。服务可能希望最终用户意识到订阅，并且必须允许最终用户撤销在直接响应用户操作而创建的订阅。

### 14.4调用顺序

订阅 firehose 的调用顺序必须遵循下图。它显示了应用程序和订阅的手动注册，然后是最终用户使用服务的API之一。在流程的这一部分，必须存储两件事：

1. 服务必须存储最终用户同意从该特定应用程序接收通知的行为（通常是后台使用OAUTH范围）。
2. 订阅应用程序必须存储最终用户的令牌，以便在收到更改通知后回调详细信息。

序列的最后一部分是通知流本身。

非规范实施指南：服务中的资源发生更改，并且该服务需要运行以下逻辑：

1. 确定有权访问资源的用户集合，从而可以期望应用程序代表他们接收有关资源的通知。
2. 查看哪些用户已同意接收通知，以及从哪些应用程序。
3. 查看哪些应用程序已注册firehose订阅。
4. 加入1、2、3以产生必须发送到应用程序的具体通知集。

应该注意的是，用户同意的行为和建立firehose订阅的行为可以以任何顺序到达。服务应发送通知，并以两种顺序处理设置。

[![Firehose订阅设置](https://camo.githubusercontent.com/df0495791d84a25a2285a6cc1071cafba58efcef/687474703a2f2f7777772e77656273657175656e63656469616772616d732e636f6d2f6367692d62696e2f63647261773f6c7a3d626d39305a534276646d56794945526c646d56736233426c63697767515856306232316864476c76626977675158427749464e6c636e5a6c636a6f67436941674943416751573441454155414a776b6762476c725a53424e62335a705a55316861325679414341475632467564484d6764473867615735305a5764795958526c494864706447676763484a70625746796553427a5a584a3261574e6c4144634752484a7663474a766541706c626d5167626d39305a516f416751774c5169425162334a3059577773494552434149454a42564a6c5a326c7a6448494167526b4852454967546d393061575a70597743424c41567a414345476458526f4143734655774267426a6f675647686c414630654149465f436b4e736157567564414174426d56755a4342316332567963796367596e4a7664334e6c6369427663694270626e4e30595778735a5751675958427743674342495177416769516741494641425143424953384167516f4749446f6754574675645746734149467a45516f4b4367434441676f384c53302d4149497143694136494578765a326c7549476c7564473841676a384a41494931454342565743414b41436f4b4c5434674b77434357424d364149514742553568625755675a58526a4c6743444651344147784a446232356d61584a74414945424345466a5932567a637942556232746c62676f4b41494d33457941745069417441494e6b4351426e426b6c454149454d43774342565155416851494d41495233436d4e7663476c6c63774172434143434948414168484d4d41494d4b44774344414267364948646c596d6876623273676367434365673441676e55534149565144546f41685859485a584941677767474149635442674245435656535443776755324e7663475541687a49475355514b54674347505177416868774e4941434442683441486845416778455062674342616777416778774e41494d61446941416778304d6257463549474e7663486b414c5245416856747141495a484230463164476876636d6c36414959374277434758516374506941724149457544564a6c6358566c63335167595143464f515a306279424551694277636d39305a574e305a5751676157356d62334941696951474367434442517374506941744149637443564a6c5a476c795a574e30414459484147774e494756755a4842766157353041496f57426d45414477317941485947414945514441434a564163415377746c5a41415948674349434167414d4163416341344168476f474145304641494564466d4a6859327367644738416846384e6158526f49474e765a47554167686f61615143426167634167546f484144304a4149492d42334d4150677341676c4548414573464149497a4467434258773041676e3847644739725a57357a414363534149305f42584a705a3268304947396d4149747044554e685932686c4948526f5958516764476870637942566332567949456c454948427962335a705a47566b41494e4e437743495a676f416767634a41494e3744334e776232357a4149305f427743454367597349484a6c5a6e4a6c63326773494746755a43424a524143424841634167514d5041495941445143424441634167555547596e6b416a466b4649456c454149516b47335231636d34416846344d4948527649474d416a523846414977526167434a5677314762473933414959714351434d61516741676d6f4b614746755a3255416a3359464149465842575268644745674c534230655842705932467349485a70595143514467567959574e306157356e414a41504267434a51517432615745416a6e7348436743504e676f67414968444541434b5a7730416b464d4641496b424477434444415541676b59574b77424e437743485741706a4149457942514348526730416857554859574e6f414951654441434566775668626d5167496e4e70626d4e6c49674346455159416b53514f414952334367434e66776341684851464149705145414342556773416846416341494938425746755a4342755a58634159525141684655544f6942566347526864475567633352686448554167536b47414946444251417845776f4b436726733d6d736367656e)](https://camo.githubusercontent.com/df0495791d84a25a2285a6cc1071cafba58efcef/687474703a2f2f7777772e77656273657175656e63656469616772616d732e636f6d2f6367692d62696e2f63647261773f6c7a3d626d39305a534276646d56794945526c646d56736233426c63697767515856306232316864476c76626977675158427749464e6c636e5a6c636a6f67436941674943416751573441454155414a776b6762476c725a53424e62335a705a55316861325679414341475632467564484d6764473867615735305a5764795958526c494864706447676763484a70625746796553427a5a584a3261574e6c4144634752484a7663474a766541706c626d5167626d39305a516f416751774c5169425162334a3059577773494552434149454a42564a6c5a326c7a6448494167526b4852454967546d393061575a70597743424c41567a414345476458526f4143734655774267426a6f675647686c414630654149465f436b4e736157567564414174426d56755a4342316332567963796367596e4a7664334e6c6369427663694270626e4e30595778735a5751675958427743674342495177416769516741494641425143424953384167516f4749446f6754574675645746734149467a45516f4b4367434441676f384c53302d4149497143694136494578765a326c7549476c7564473841676a384a41494931454342565743414b41436f4b4c5434674b77434357424d364149514742553568625755675a58526a4c6743444651344147784a446232356d61584a74414945424345466a5932567a637942556232746c62676f4b41494d33457941745069417441494e6b4351426e426b6c454149454d43774342565155416851494d41495233436d4e7663476c6c63774172434143434948414168484d4d41494d4b44774344414267364948646c596d6876623273676367434365673441676e55534149565144546f41685859485a584941677767474149635442674245435656535443776755324e7663475541687a49475355514b54674347505177416868774e4941434442683441486845416778455062674342616777416778774e41494d61446941416778304d6257463549474e7663486b414c5245416856747141495a484230463164476876636d6c36414959374277434758516374506941724149457544564a6c6358566c63335167595143464f515a306279424551694277636d39305a574e305a5751676157356d62334941696951474367434442517374506941744149637443564a6c5a476c795a574e30414459484147774e494756755a4842766157353041496f57426d45414477317941485947414945514441434a564163415377746c5a41415948674349434167414d4163416341344168476f474145304641494564466d4a6859327367644738416846384e6158526f49474e765a47554167686f61615143426167634167546f484144304a4149492d42334d4150677341676c4548414573464149497a4467434258773041676e3847644739725a57357a414363534149305f42584a705a3268304947396d4149747044554e685932686c4948526f5958516764476870637942566332567949456c454948427962335a705a47566b41494e4e437743495a676f416767634a41494e3744334e776232357a4149305f427743454367597349484a6c5a6e4a6c63326773494746755a43424a524143424841634167514d5041495941445143424441634167555547596e6b416a466b4649456c454149516b47335231636d34416846344d4948527649474d416a523846414977526167434a5677314762473933414959714351434d61516741676d6f4b614746755a3255416a3359464149465842575268644745674c534230655842705932467349485a70595143514467567959574e306157356e414a41504267434a51517432615745416a6e7348436743504e676f67414968444541434b5a7730416b464d4641496b424477434444415541676b59574b77424e437743485741706a4149457942514348526730416857554859574e6f414951654441434566775668626d5167496e4e70626d4e6c49674346455159416b53514f414952334367434e66776341684851464149705145414342556773416846416341494938425746755a4342755a58634159525141684655544f6942566347526864475567633352686448554167536b47414946444251417845776f4b436726733d6d736367656e)

对于每个用户的订阅，应用程序注册可以手动或自动进行。每个用户订阅的调用流程必须遵循下图。它显示了最终用户使用服务的API之一，并且同样，必须存储相同的两件事：

1. 服务必须存储最终用户同意从该特定应用程序接收通知的行为（通常是后台使用OAUTH范围）。
2. 订阅应用程序必须存储最终用户的令牌，以便在收到更改通知后回调详细信息。

在这种情况下，使用来自订阅应用程序的最终用户令牌以编程方式设置订阅。应用程序必须将注册订阅的ID与用户令牌一起存储。

非规范性实施指南：在序列的最后部分，当服务中的数据项发生更改并且服务需要运行以下逻辑时：

1. 查找通过资源与已更改数据相对应的一组预订。
2. 对于使用app + user令牌创建的订阅，请使用订阅创建者的订阅ID和用户ID向每个订阅发送通知给应用。

- 对于使用仅应用令牌创建的订阅，请检查已更改数据的所有者或具有已更改数据可见性的任何用户均已同意向该应用发送通知，如果是，则按用户ID向该应用发送一组通知带有订阅ID的订阅。

  [![用户订阅设置](https://camo.githubusercontent.com/de3129ed4573bcc492e23ba013ba4c370d00facf/687474703a2f2f7777772e77656273657175656e63656469616772616d732e636f6d2f6367692d62696e2f63647261773f6c7a3d626d39305a534276646d56794945526c646d56736233426c63697767515856306232316864476c76626977675158427749464e6c636e5a6c636a6f67436941674943416751573441454155414a776b6762476c725a53424e62335a705a55316861325679414341475632467564484d6764473867615735305a5764795958526c494864706447676763484a70625746796553427a5a584a3261574e6c4144634752484a7663474a766541706c626d5167626d39305a516f416751774c5169425162334a3059577773494552434149454a42564a6c5a326c7a6448494167526b4852454967546d393061575a70597743424c41567a414345476458526f4143734655774267426a6f675647686c414630654149465f436b4e736157567564414174426d56755a4342316332567963796367596e4a7664334e6c6369427663694270626e4e30595778735a5751675958427743674342495177416769516741494641425143424953384167516f4749446f416757775243677068624851416779554941494548426942794142514d494341416778734c50433074506743445477733649454e76626d5a705a3356795a516f6749414344614173674c5434674b77434357424d4165675a4f5957316c49475630597934416841674641494d61445141664567426442584a744149515f4255466a5932567a637942556232746c41494554426743444f7849674c5434674c5143424667784263484167535551416848774959334a6c644143424778417450674346466773674f69424662574a6c5a41416b46475673633255675457467564574673414949454a41434562516b674f69424d6232647062694270626e527641495542435143424b52465657414347474155414c516f416768386d4149495a4b77434243416341676a6f4e41494973484143474c776b41676a38494149455344674345434159416831454c41496446436d4e7663476c6c63774175434756755a41434565476f4168575148515856306147397961586f4168563848414956364279302d494373416732414e556d56786457567a6443426841495256426e527649455243494842796233526c5933526c5a434270626d5a766367434a5151594b414951614379302d49433041686b6f4a556d566b61584a6c593351414e676341624130675a57356b63473970626e514169544d475951415044584941646759416752414d414968784277424c4332566b414267654149526a43414177423045416351785657416f415351674167527757596d466a617942306277434664417741696c77465932396b5a5143434752707041494670427743424f51634150516b41676a30486377412d437743435541634153775541676a494f414946654451434366675a306232746c626e4d414a7849416a467346636d6c6e61485167623259416977554e5132466a61475567644768686443423061476c7a4946567a5a5849675355516763484a76646d6c6b5a5751416730774c41495536427743434241774167336f5063334276626e4d416a4673484149514a42697767636d566d636d567a614377675957356b49456c454149456342774342417738416944454e4149454d4277434252515a696551434c645155675355514168434d62644856796267434558517767644738675977434d4f77554b4367434c4c326f416a58554d41497754447743504e516f74506973416a6877514f67434f5251646c6367434d56775941673359495a574a6f62323972494656535443776755324e76634755416b414547535551416a776f4f41493572445341416932554b41494e464251434c5977304148424541677a554f4f6942754149453244414267434143444342316f5a5143426151354a52414344597755416168494167684234526d78766477434a4d776b416a45304941495630436d6868626d646c414a4963425143455951566b595852684943306764486c7761574e6862434232615745416b6a5146636d466a64476c755a7743534e5159416a56384c646d6c68414a456842776f416b56774b4941434e666841416841734e414a4a35425143435751384168685946414956514669734154517341696d454b597743424d675541696b384e414968764232466a614143484b41774169416b465957356b49434a7a6157356a5a53494169427347414a4e4b4467434941516f4168423063414946534377434857687741676a77465957356b4947356c647742684641434858784d36494656775a4746305a53427a64474630645143424b51594167554d464144455443676f4b26733d6d736367656e)](https://camo.githubusercontent.com/de3129ed4573bcc492e23ba013ba4c370d00facf/687474703a2f2f7777772e77656273657175656e63656469616772616d732e636f6d2f6367692d62696e2f63647261773f6c7a3d626d39305a534276646d56794945526c646d56736233426c63697767515856306232316864476c76626977675158427749464e6c636e5a6c636a6f67436941674943416751573441454155414a776b6762476c725a53424e62335a705a55316861325679414341475632467564484d6764473867615735305a5764795958526c494864706447676763484a70625746796553427a5a584a3261574e6c4144634752484a7663474a766541706c626d5167626d39305a516f416751774c5169425162334a3059577773494552434149454a42564a6c5a326c7a6448494167526b4852454967546d393061575a70597743424c41567a414345476458526f4143734655774267426a6f675647686c414630654149465f436b4e736157567564414174426d56755a4342316332567963796367596e4a7664334e6c6369427663694270626e4e30595778735a5751675958427743674342495177416769516741494641425143424953384167516f4749446f416757775243677068624851416779554941494548426942794142514d494341416778734c50433074506743445477733649454e76626d5a705a3356795a516f6749414344614173674c5434674b77434357424d4165675a4f5957316c49475630597934416841674641494d61445141664567426442584a744149515f4255466a5932567a637942556232746c41494554426743444f7849674c5434674c5143424667784263484167535551416848774959334a6c644143424778417450674346466773674f69424662574a6c5a41416b46475673633255675457467564574673414949454a41434562516b674f69424d6232647062694270626e527641495542435143424b52465657414347474155414c516f416768386d4149495a4b77434243416341676a6f4e41494973484143474c776b41676a38494149455344674345434159416831454c41496446436d4e7663476c6c63774175434756755a41434565476f4168575148515856306147397961586f4168563848414956364279302d494373416732414e556d56786457567a6443426841495256426e527649455243494842796233526c5933526c5a434270626d5a766367434a5151594b414951614379302d49433041686b6f4a556d566b61584a6c593351414e676341624130675a57356b63473970626e514169544d475951415044584941646759416752414d414968784277424c4332566b414267654149526a43414177423045416351785657416f415351674167527757596d466a617942306277434664417741696c77465932396b5a5143434752707041494670427743424f51634150516b41676a30486377412d437743435541634153775541676a494f414946654451434366675a306232746c626e4d414a7849416a467346636d6c6e61485167623259416977554e5132466a61475567644768686443423061476c7a4946567a5a5849675355516763484a76646d6c6b5a5751416730774c41495536427743434241774167336f5063334276626e4d416a4673484149514a42697767636d566d636d567a614377675957356b49456c454149456342774342417738416944454e4149454d4277434252515a696551434c645155675355514168434d62644856796267434558517767644738675977434d4f77554b4367434c4c326f416a58554d41497754447743504e516f74506973416a6877514f67434f5251646c6367434d56775941673359495a574a6f62323972494656535443776755324e76634755416b414547535551416a776f4f41493572445341416932554b41494e464251434c5977304148424541677a554f4f6942754149453244414267434143444342316f5a5143426151354a52414344597755416168494167684234526d78766477434a4d776b416a45304941495630436d6868626d646c414a4963425143455951566b595852684943306764486c7761574e6862434232615745416b6a5146636d466a64476c755a7743534e5159416a56384c646d6c68414a456842776f416b56774b4941434e666841416841734e414a4a35425143435751384168685946414956514669734154517341696d454b597743424d675541696b384e414968764232466a614143484b41774169416b465957356b49434a7a6157356a5a53494169427347414a4e4b4467434941516f4168423063414946534377434857687741676a77465957356b4947356c647742684641434858784d36494656775a4746305a53427a64474630645143424b51594167554d464144455443676f4b26733d6d736367656e)

### 14.5验证订阅

当订阅以编程方式更改或响应通过管理UI门户的更改时，需要保护订阅服务免受恶意或意外调用的干扰，因为这些服务可能会导致大量通知流量。

对于所有订阅，无论是firehose还是按用户，服务都必须在发送任何其他通知之前，通过门户网站UI或API请求发送验证请求，作为创建或修改的一部分。

验证请求的格式必须为订阅的*notificationUrl*的HTTP / HTTPS POST格式。

```none
POST https://{notificationUrl}?validationToken={randomString}
ClientState: clientOriginatedOpaqueToken (if provided by client on subscription-creation)
Content-Length: 0
```

对于要建立的订阅，应用程序必须对该请求做出200 OK的响应，并且*validationToken*值为唯一的实体主体。请注意，如果*notificationUrl*包含查询参数，则*validationToken*参数必须附加一个`&`。

如果任何质询请求在发送请求后的5秒钟内未收到规定的响应，则该服务务必返回错误，不得创建订阅，并且不得将其他请求或通知发送给*notificationUrl*。

服务可以对URL所有权执行其他验证。

### 14.6接收通知

服务应该发送通知以响应服务数据的更改，这些更改不包括更改本身的详细信息，但应包含足够的信息，以便订阅应用程序可以适当地响应以下过程：

1. 应用程序必须标识正确的缓存OAuth令牌以用于回调
2. 应用程序可以查找任何先前的delta令牌以获取相关的更改范围
3. 应用程序必须确定要调用的URL，以对服务的新状态执行相关查询，该查询可以是增量查询。

提供通知的服务将被中继给最终用户，可以选择在通知包中添加更多细节，以减少其服务上的传入调用负载。此类服务必须明确指出，不能保证一定会发送通知，并且通知可能有损或乱序。

通知可以汇总并分批发送。应用程序必须准备好在单个推送通知中接收多个事件。

该服务务必将所有Web Hook数据通知作为POST请求发送。

服务必须允许30秒钟的通知超时时间。如果发生超时或应用程序以5xx响应进行响应，则服务应使用指数退避重试通知。所有其他响应将被忽略。

服务不得遵循301/302重定向请求。

#### 14.6.1通知有效负载

通知有效负载的基本格式是事件列表，每个事件包含订阅的ID（已更改），该订阅的参考资源已更改，更改的类型，应消耗的资源来标识更改的确切详细信息以及足够的标识信息以供查看调用该资源所需的令牌。

对于firehose订阅，具体示例如下所示：

```json
{
  "value": [
    {
      "subscriptionId": "32b8cbd6174ab18b",
      "resource": "https://api.contoso.com/v1.0/users/user@contoso.com/files?$delta",
      "userId" : "<User GUID>",
      "tenantId" : "<Tenant Id>"
    }
  ]
}
```

对于每个用户的订阅，其具体示例如下所示：

```json
{
  "value": [
    {
      "subscriptionId": "32b8cbd6174ab183",
      "clientState": "clientOriginatedOpaqueToken",
      "expirationDateTime": "2016-02-04T11:23Z",
      "resource": "https://api.contoso.com/v1.0/users/user@contoso.com/files/$delta",
      "userId" : "<User GUID>",
      "tenantId" : "<Tenant Id>"
    },
    {
      "subscriptionId": "97b391179fa22",
      "clientState ": "clientOriginatedOpaqueToken",
      "expirationDateTime": "2016-02-04T11:23Z",
      "resource": "https://api.contoso.com/v1.0/users/user@contoso.com/files/$delta",
      "userId" : "<User GUID>",
      "tenantId" : "<Tenant Id>"
    }
  ]
}
```

以下是JSON有效负载的详细说明。

通知项包含一个顶层对象，该对象包含一系列事件，每个事件都标识了由于发送此通知而导致的订阅。

| 领域 | 描述                                       |
| ---- | ------------------------------------------ |
| value   | 自上次通知以来在订阅范围内引发的事件数组。 |

事件数组的每一项都包含以下属性：

| 领域               | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| subscriptionId     | 已发送此通知的订阅的ID。 服务必须提供*subscriptionId*字段。  |
| clientState        | 如果在创建订阅时提供了*clientState*字段，则服务必须提供*clientState*字段。 |
| expirationDateTime | 如果订阅有一个，则服务必须提供*expirationDateTime*字段。     |
| resource               | 服务必须提供资源字段。订阅应用程序必须将该URL视为不透明。在更丰富的通知的情况下，它可以包含在消息内容中，该消息内容隐式包含资源URL，以避免重复。 如果服务将此数据作为更详细的数据包的一部分提供，则无需复制。 |
| userId           | 服务必须为用户范围的资源提供此字段。对于用户范围的资源，应使用用户的唯一标识符。 在一组特定用户之间共享资源的情况下，必须发送多个通知，并传递每个用户的唯一标识符。 对于租户范围内的资源，应使用预订的用户ID。 |
| tenantId           | 希望支持跨租户请求的服务应提供此字段。提供有关租户范围数据的通知的服务必须发送此字段。 |

### 14.7以编程方式管理订阅

对于每个用户的订阅，必须提供一个API以创建和管理订阅。API必须至少支持此处描述的操作。

#### 14.7.1创建订阅

客户端通过对订阅资源发出POST请求来创建订阅。订阅名称空间是通过POST操作由客户端定义的。

```none
https://api.contoso.com/apiVersion/$subscriptions
```

POST请求包含一个要创建的订阅对象。该订阅对象具有以下属性：

| 物业名称         | 需要 | 笔记                                                         |
| --------------- | ---- | ------------------------------------------------------------ |
| resource        | 是   | 要观看的资源路径。                                           |
| notificationUrl | 是   | 目标Web挂钩URL。                                             |
| clientState     | 否   | 不透明字符串在所有通知上传递回客户端。调用者可以选择使用它来提供标记机制。 |

如果成功创建了订阅，则服务必须使用状态代码201 CREATED和至少包含以下属性的主体进行响应：

| 物业名称           | 需要 | 笔记                                             |
| ------------------ | ---- | ------------------------------------------------ |
| ID                 | 是   | 新订阅的唯一ID，以后可用于更新/删除订阅。        |
| expirationDateTime | 否   | 使用现有的Microsoft REST API准则定义的时间格式。 |

订阅的创建应该是幂等的。范围限定于auth令牌的属性的组合提供了唯一性约束。

以下是使用用户+应用程序主体订阅文件通知的示例请求：

```none
POST https://api.contoso.com/files/v1.0/$subscriptions HTTP 1.1
Authorization: Bearer {UserPrincipalBearerToken}

{
  "resource": "http://api.service.com/v1.0/files/file1.txt",
  "notificationUrl": "https://contoso.com/myCallbacks",
  "clientState": "clientOriginatedOpaqueToken"
}
```

服务应该以最小的响应格式响应这样的消息：

```json
{
  "id": "32b8cbd6174ab18b",
  "expirationDateTime": "2016-02-04T11:23Z"
}
```

下面是使用仅应用程序主体的示例，其中应用程序正在监视其授权的所有文件：

```none
POST https://api.contoso.com/files/v1.0/$subscriptions HTTP 1.1
Authorization: Bearer {ApplicationPrincipalBearerToken}

{
  "resource": "All.Files",
  "notificationUrl": "https://contoso.com/myCallbacks",
  "clientState": "clientOriginatedOpaqueToken"
}
```

服务应该以最小的响应格式响应这样的消息：

```json
{
  "id": "8cbd6174abb391179",
  "expirationDateTime": "2016-02-04T11:23Z"
}
```

#### 14.7.2更新订阅

服务可以支持修改订阅。为了更新现有订阅的属性，客户端使用PATCH请求来提供ID和需要更改的属性。忽略的属性将保留其值。要删除属性，请为其分配JSON null值。

与创建一样，订阅是单独管理的。

以下请求更改现有订阅的通知URL：

```none
PATCH https://api.contoso.com/files/v1.0/$subscriptions/{id} HTTP 1.1
Authorization: Bearer {UserPrincipalBearerToken}

{
  "notificationUrl": "https://contoso.com/myNewCallback"
}
```

如果PATCH请求包含一个新的*notificationUrl*，则服务器必须如上所述对其进行验证。如果新的URL验证失败，则该服务务必使PATCH请求失败，并使订阅保持其先前状态。

服务必须返回一个空的主体并`204 No Content`指示补丁成功。

如果补丁失败，服务必须返回错误体和状态码。

该操作必须成功完成或失败。

#### 14.7.3删除订阅

服务必须支持删除订阅。可以通过对订阅资源发出DELETE请求来删除现有订阅：

```none
DELETE https://api.contoso.com/files/v1.0/$subscriptions/{id} HTTP 1.1
Authorization: Bearer {UserPrincipalBearerToken}
```

与更新一样，服务必须返回`204 No Content`以成功删除，或者返回错误体和状态码以指示失败。

#### 14.7.4 枚举订阅

为了获取活动订阅的列表，客户端使用用户+应用程序或仅应用程序承载令牌对订阅资源发出GET请求：

```none
GET https://api.contoso.com/files/v1.0/$subscriptions HTTP 1.1
Authorization: Bearer {UserPrincipalBearerToken}
```

服务必须使用用户+应用程序主体承载令牌返回以下格式：

```json
{
  "value": [
    {
      "id": "32b8cbd6174ab18b",
      "resource": " http://api.contoso.com/v1.0/files/file1.txt",
      "notificationUrl": "https://contoso.com/myCallbacks",
      "clientState": "clientOriginatedOpaqueToken",
      "expirationDateTime": "2016-02-04T11:23Z"
    }
  ]
}
```

可以使用仅应用程序主体承载令牌返回的示例：

```json
{
  "value": [
    {
      "id": "6174ab18bfa22",
      "resource": "All.Files ",
      "notificationUrl": "https://contoso.com/myCallbacks",
      "clientState": "clientOriginatedOpaqueToken",
      "expirationDateTime": "2016-02-04T11:23Z"
    }
  ]
}
```

### 14.8 安全性

所有服务 URL 必须为 HTTPS（也就是说，所有入站调用必须为 HTTPS ）。处理 Web Hooks 的服务必须接受 HTTPS 。

我们建议允许客户端定义的 Web Hook 回调 URL 的服务不应通过 HTTP 传输数据。这是因为信息可能会通过客户端，网络，服务器日志和其他机制无意间公开。

但是，在某些情况下，由于客户端端点或软件限制，无法遵循上述建议。因此，服务可以允许 Web 钩 URL 为 HTTP 。

此外，允许客户端定义的 HTTP Web 挂钩回调 URL 的服务应符合工程领导指定的隐私策略。这通常包括建议客户端选择SSL连接并遵守特殊的预防措施，以确保正确处理日志和其他服务数据收集。

例如，服务可能不希望要求开发人员在机上生成证书。服务可能仅在测试帐户上启用此功能
