---
title: HTTP Cookie 完全指南：原理、属性与安全
published: 2026-04-06
description: 深入理解 HTTP Cookie：会话管理/个性化/跟踪三大用途、Set-Cookie 与 Cookie 标头、生命周期（Expires/Max-Age）、Secure 与 HttpOnly、Domain/Path/SameSite 作用域、Cookie 安全前缀与安全最佳实践。
image: ''
tags: [HTTP, Cookie, 前端基础, 网络安全]
category: 'HTTP协议'
draft: false
lang: zh-cn
toc: true
---

> 本文整理自 MDN 官方文档《HTTP Cookie》，梳理 Cookie 的原理、属性、安全与隐私问题。

## 一、HTTP Cookie 是什么

**HTTP Cookie**（也叫 Web Cookie 或浏览器 Cookie）是**服务器发送到用户浏览器并保存在本地的一小块数据**。

浏览器会存储 Cookie，并在下次向同一服务器再发起请求时**携带并发送到服务器**。通常它用于告知服务端两个请求是否来自**同一浏览器**——如保持用户的登录状态。**Cookie 使基于无状态的 HTTP 协议记录稳定的状态信息成为可能。**

Cookie 主要用于以下三个方面：

1. **会话状态管理**——如用户登录状态、购物车、游戏分数或其他需要记录的信息
2. **个性化设置**——如用户自定义设置、主题和其他设置
3. **浏览器行为跟踪**——如跟踪分析用户行为等

:::warning
Cookie 曾一度是客户端数据存储的唯一手段，但**现在推荐使用现代存储 API**。由于服务器指定 Cookie 后，浏览器的**每次请求都会携带 Cookie 数据**，会带来额外的性能开销（尤其是在移动环境下）。新的浏览器 API 已经允许开发者直接将数据存储到本地，如 **Web Storage API（localStorage 和 sessionStorage）** 或 **IndexedDB**。
:::

## 二、创建 Cookie：Set-Cookie 和 Cookie 标头

服务器收到 HTTP 请求后，可以在**响应标头**中添加一个或多个 **Set-Cookie** 选项；浏览器收到响应后通常会保存下 Cookie，并将其放在 **HTTP Cookie 标头**内，向同一服务器发出请求时一起发送。你可以指定过期日期或时间段、对指定的域和路径设置额外限制，以控制 Cookie 的发送范围。

:::tip
**Set-Cookie** 是 HTTP 响应标头，用于将 Cookie 从服务器发送到用户代理，以便用户代理在后续请求中将其发送回服务器。要发送多个 Cookie，应在同一响应中发送**多个 Set-Cookie 标头**。
:::

一个简单的 Cookie 长这样：

```http
Set-Cookie: <cookie-name>=<cookie-value>
```

服务器通过响应标头告知客户端存储一对 Cookie：

```http
HTTP/1.0 200 OK
Content-type: text/html
Set-Cookie: yummy_cookie=choco
Set-Cookie: tasty_cookie=strawberry

[页面内容]
```

之后，对该服务器发起的每一次新请求，浏览器都会将之前保存的 Cookie 信息通过 **Cookie 请求头部**再发送给服务器：

```http
GET /sample_page.html HTTP/1.1
Host: www.example.org
Cookie: yummy_cookie=choco;tasty_cookie=strawberry
```

## 三、Cookie 的生命周期

Cookie 的生命周期可以通过两种方式定义：

- **会话期 Cookie**——在当前的会话结束之后删除。浏览器定义了"当前会话"结束的时间，一些浏览器重启时会使用**会话恢复**，这可能导致会话 Cookie 无限延长
- **持久性 Cookie**——在过期时间（**Expires**）指定的日期或有效期（**Max-Age**）指定的一段时间后被删除

```http
Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT;
```

:::warning
**会话固定攻击（session fixation）防护**：如果站点对用户进行身份验证，则每当用户进行身份验证时，都应**重新生成并重新发送会话 Cookie**（即使是已经存在的会话 Cookie），以防止第三方重用用户的会话。
:::

## 四、限制访问 Cookie：Secure 和 HttpOnly

有两种方法可以确保 Cookie 被安全发送，并且不会被意外的参与者或脚本访问：**Secure 属性**和 **HttpOnly 属性**。

**Secure 属性**

- 标记为 Secure 的 Cookie **只应通过被 HTTPS 协议加密过的请求**发送给服务端
- 它永远不会使用不安全的 HTTP 发送（本地主机除外），这意味着**中间人攻击者**无法轻松访问它
- 不安全的站点（在 URL 中带有 `http:`）无法使用 Secure 属性设置 Cookie
- 但 Secure **不会阻止对 Cookie 中敏感信息的访问**——有权访问客户端硬盘的人（或未设置 HttpOnly 时的 JavaScript）仍可以读取和修改它

**HttpOnly 属性**

- JavaScript 的 **Document.cookie API 无法访问**带有 HttpOnly 属性的 Cookie，此类 Cookie 仅作用于服务器
- 例如，持久化服务器端会话的 Cookie 不需要对 JavaScript 可用，而应具有 HttpOnly 属性
- 此预防措施有助于缓解**跨站点脚本（XSS）攻击**

```http
Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT; Secure; HttpOnly
```

## 五、定义 Cookie 发送的位置：Domain / Path / SameSite

**Domain 和 Path 标识**定义了 Cookie 的**作用域**，即允许 Cookie 发送给哪些 URL。

### 1. Domain 属性

Domain 指定了哪些主机可以接受 Cookie：

- 不指定时，默认为同一 host 设置 Cookie，**不包含子域名**
- 指定了 Domain 时，**一般包含子域名**，限制更少；当子域需要共享有关用户的信息时，这会有所帮助

:::tip
例如，如果设置 `Domain=mozilla.org`，则 Cookie 也包含在子域名中（如 `developer.mozilla.org`）。
:::

### 2. Path 属性

Path 属性指定了一个**必须存在于请求 URL 中**的路径，以便发送 Cookie 标头。以字符 `/`（%x2F）作为路径分隔符，并且**子路径也会被匹配**。

例如，设置 `Path=/docs`，以下地址都会匹配：

```
/docs
/docs/
/docs/Web/
/docs/Web/HTTP
```

但以下地址不会匹配：

```
/
/docsets
/fr/docs
```

### 3. SameSite 属性

SameSite 属性允许服务器指定**是否/何时通过跨站点请求发送 Cookie**（站点由注册的域和方案定义：http 或 https），提供针对**跨站点请求伪造攻击（CSRF）**的保护。它采用三个可能的值：**Strict、Lax 和 None**：

- **Strict**——Cookie 仅发送到它来源的站点
- **Lax**——与 Strict 相似，但在用户导航到 Cookie 的源站点时（如通过跟踪来自外部站点的链接）会发送 Cookie
- **None**——浏览器会在同站请求和跨站请求下继续发送 Cookie，但**仅在安全的上下文中**（即如果 `SameSite=None`，则还必须设置 **Secure** 属性，否则浏览器会拒绝该 Cookie）

## 六、SameSite 大白话讲解

**SameSite 就是浏览器给 Cookie 加的一个「跨站访问门禁」。**

- 你在 A 网站登录后，浏览器会存 A 网站的 Cookie（相当于你的「家门钥匙」）
- 正常情况：你访问 A 网站，浏览器自动把钥匙给 A，没问题
- 风险情况：你点了一个恶意网站 B 的链接，B 偷偷给 A 网站发请求，如果浏览器把 A 的钥匙给了 B，B 就能冒充你操作 A 网站（这就是 **CSRF 攻击**）
- **SameSite 的作用**：规定「什么时候能把钥匙给别人」，防止坏人偷用你的钥匙

### 1. Strict（最严：只给自家，外人一概不给）

> 人话翻译：我的钥匙，只有我自己开门能用，别人碰都别想碰。

- **规则**：只有「完全同站」的请求才会带 Cookie
  - 同站：域名、协议（http/https）完全一样，比如你在 `https://a.com` 访问 `https://a.com/xxx`
  - 跨站：从 `https://b.com` 跳转到 `https://a.com`，哪怕是点链接，也绝对不带
- **生活类比**：你家门钥匙，只有你自己掏出来开门能用，朋友、快递员、陌生人谁都不能用你的钥匙开门
- **适用场景**：银行、支付、核心账号操作等极高安全要求的场景
- **缺点**：太严了！比如你在微信里点朋友发的银行链接，打开后会直接显示「未登录」，因为 Strict 不让跨站带 Cookie

### 2. Lax（宽松：自家能用，安全的跨站也能用，危险的不给）

> 人话翻译：我的钥匙，自己开门能用，正常的「点链接跳转」也能用，但偷偷摸摸的请求绝对不给。

- **规则**：同站请求必带，安全的跨站请求（GET 跳转）带，危险的跨站请求（POST 提交、iframe 嵌入、AJAX 请求）绝对不带
  - 允许带：你点了 `https://b.com` 页面里的 `<a href="https://a.com">` 链接，跳转到 A 网站，会带 Cookie（正常的用户主动跳转）
  - 禁止带：B 网站偷偷用 `<form action="https://a.com/transfer" method="POST">` 给 A 发转账请求，绝对不带 Cookie，转账失败
- **生活类比**：自己开门能用，朋友给你递钥匙（正常点链接）也能用，但陌生人偷偷配钥匙、撬锁（偷偷发请求）绝对用不了
- **适用场景**：绝大多数网站的默认选择（现在浏览器已经把 Lax 设为 Cookie 的默认 SameSite 值）
- **优点**：安全 + 体验平衡，既防 CSRF，又不影响正常的跨站跳转登录

### 3. None（无限制：谁要都给，必须加 Secure）

> 人话翻译：我的钥匙，谁要都给，但必须走「安全通道」（HTTPS）。

- **规则**：不管是同站还是跨站，所有请求都带 Cookie，但强制要求必须是 HTTPS 协议（必须加 Secure 属性，否则浏览器直接拒绝）
- **生活类比**：谁要都给，但只能在「有保安的安全通道」里递钥匙，绝对不能在大街上随便给
- **适用场景**：必须跨站共享 Cookie 的场景，比如：
  - 第三方登录（微信 / QQ 登录第三方网站）
  - 跨域 iframe 嵌入（比如把 A 网站的支付页面嵌到 B 网站里）
  - 前后端分离的跨域请求（但现在更推荐用 JWT 存 Cookie 加 Lax/Strict）
- **风险**：如果不加 Secure，等于把钥匙随便给所有人，CSRF 风险极高；现在浏览器已经不允许 HTTP 下用 `SameSite=None`

```http
Set-Cookie: mykey=myvalue; SameSite=Strict
```

## 七、Cookie 前缀：__Host- 与 __Secure-

Cookie 的机制使得服务器无法确认 Cookie 是在**安全来源**上设置的，甚至无法确定 Cookie 最初是在哪里设置的。子域上的易受攻击的应用程序可以使用 Domain 属性设置 Cookie，从而访问所有其他子域上的该 Cookie，在**会话劫持攻击**中可能会被滥用。

- **`__Host-` 前缀**——仅当 Cookie 同时满足以下条件时，才在 Set-Cookie 标头中被接受：用 **Secure** 属性标记、从**安全来源**发送、**不包括 Domain 属性**、**Path 属性设置为 `/`**。这样这些 Cookie 可以被视为"domain-locked"（域锁定）
- **`__Secure-` 前缀**——仅当 Cookie 用 **Secure** 属性标记且从安全来源发送时，才在 Set-Cookie 标头中被接受。该前缀限制**弱于 `__Host-` 前缀**

:::tip
带有这些前缀的 Cookie，如果不符合限制会被浏览器**拒绝**。这确保了如果子域要创建带前缀的 Cookie，它要么局限于该子域，要么被完全忽略。由于应用服务器仅在确定用户是否已通过身份验证或 CSRF 令牌是否正确时才检查特定的 Cookie 名称，这有效地充当了针对**会话劫持**的防御措施。
:::

## 八、JavaScript 访问 Cookie：Document.cookie

通过 **Document.cookie** 属性可创建新的 Cookie。如果未设置 HttpOnly 标记，你也可以从 JavaScript 访问现有的 Cookie：

```js
document.cookie = "yummy_cookie=choco";
document.cookie = "tasty_cookie=strawberry";
console.log(document.cookie);
// logs "yummy_cookie=choco; tasty_cookie=strawberry"
```

:::warning
通过 JavaScript 创建的 Cookie **不能包含 HttpOnly 标志**。请留意安全章节提到的安全隐患：JavaScript 可以通过**跨站脚本攻击（XSS）** 的方式窃取 Cookie。
:::

## 九、安全

缓解涉及 Cookie 的攻击的方法：

- 使用 **HttpOnly 属性**防止通过 JavaScript 访问 Cookie 值
- 用于敏感信息（例如指示身份验证）的 Cookie 的**生存期应较短**，并且 **SameSite 属性设置为 Strict 或 Lax**。在支持 SameSite 的浏览器中，这样可确保**不与跨站点请求一起发送身份验证 Cookie**，此类请求实际上不会向应用服务器进行身份验证

## 十、跟踪和隐私：第三方 Cookie

Cookie 与特定域和方案（例如 http 或 https）相关联，如果设置了 `Set-Cookie Domain` 属性，也可能与子域相关联：

- 如果 Cookie 的域和方案**匹配**当前页面，则认为 Cookie 和页面来自同一站点，称为**第一方 Cookie（first-party cookie）**
- 如果域和方案**不同**，则称为**第三方 Cookie（third-party cookie）**

托管网页的服务器设置第一方 Cookie，但页面可能包含存储在其他域服务器上的图像或其他组件（例如广告横幅），这些组件可能会设置第三方 Cookie——它们**主要用于网络上的广告和跟踪**（例如 Google 使用的 Cookie 类型）。

:::warning
**隐私风险**：第三方服务器可以基于同一浏览器在访问多个站点时发送给它的 Cookie，建立用户浏览历史和习惯的**配置文件**。Firefox 默认会阻止已知包含跟踪器的第三方 Cookie，其他浏览器或扩展程序也可能阻止。阻止 Cookie 会导致某些第三方组件（如社交媒体小部件）无法正常运行。
:::

## 总结：Cookie 属性速查表

| 属性 | 作用 | 关键点 |
|------|------|--------|
| `Expires` / `Max-Age` | 定义生命周期 | 会话期 vs 持久性 |
| `Secure` | 仅通过 HTTPS 发送 | 防中间人窃听 |
| `HttpOnly` | 禁止 JavaScript 访问 | 防 XSS 窃取 |
| `Domain` / `Path` | 定义发送作用域 | 子域、子路径匹配 |
| `SameSite` | 控制跨站发送 | Strict / Lax / None，防 CSRF |
| `__Host-` / `__Secure-` | 安全前缀 | 强制 Secure、域锁定 |

> 资料来源：[MDN — HTTP Cookie](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies)
