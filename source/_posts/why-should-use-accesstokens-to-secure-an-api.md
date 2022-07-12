---
title: why-should-use-accesstokens-to-secure-an-api
date: 2022-07-12 10:15:32
tags: oauth2
categories: 技术
---
# 为什么应始终使用访问令牌来保护 API

[https://auth0.com/blog/why-should-use-accesstokens-to-secure-an-api/](https://auth0.com/blog/why-should-use-accesstokens-to-secure-an-api/)

网络上对OpenID Connect和OAuth 2.0规范及其各自令牌之间的差异存在很多困惑。因此，许多开发人员发布不安全的应用程序，损害了用户的安全性。标识提供者之间相互矛盾的实现也无济于事。

本文试图阐明什么是什么，并解释为什么您应该始终使用访问令牌来保护API，而不是ID令牌。


---


## 两种互补规格

OAuth 2.0 用于**授予授权**.它使您能够授权 Web 应用程序 A 从 Web 应用程序 B 访问您的信息，而无需共享您的凭据。它是用*只*授权在脑海中，并且不包含任何身份验证机制（换句话说，它不为授权服务器提供任何验证用户身份的方法）。

OpenID Connect 建立在 OAuth 2.0 之上。它使你（用户）能够验证你的身份并提供一些基本的个人资料信息，而无需共享你的凭据。

一个例子是待办事项应用程序，它允许您使用Google帐户登录，并且可以将您的待办事项作为日历条目推送到Google日历上。验证身份的部分通过 OpenID Connect 实现，而授权待办事项应用程序通过添加条目来修改日历的部分则通过 OAuth 2.0 实现。

>OpenID Connect是关于某人是谁的。OAuth 2.0是关于他们被允许做什么。
您可能会注意到*“无需共享您的凭据”*部分，在我们之前对两个规范的定义中。在这两种情况下，您共享的都是令牌。

OpenID Connect 会发出一个标识令牌，称为 ，而 OAuth 2.0 会发出 .通过免费的 OpenID Connect 手册了解有关 OIDC 的更多信息：`id_tokenaccess_token`

## 如何使用每个令牌

这是一个 [JWT](https://auth0.com/docs/jwt)，仅供客户端使用。在我们之前使用的示例中，当您使用Google进行身份验证时，会从Google发送到待办事项应用程序，该应用程序表示您是谁。待办事项应用程序可以分析[令牌的内容](https://openid.net/specs/openid-connect-core-1_0.html#StandardClaims)，并使用此信息（如您的姓名和个人资料图片）来自定义用户体验。`id_tokenid_token`

**注意：**除非您已验证，否则切勿在 中使用该信息！有关详细信息，请参阅：[如何验证 ID 令牌](https://auth0.com/docs/tokens/id-token#how-to-validate-an-id-token)。有关可用于验证 JWT 的库列表，请参阅 [JWT.io](https://jwt.io/)。`id_token`

这`access_token`可以是任何类型的令牌（不一定是 JWT），并且适用于 API。其目的是通知 API 此令牌的持有者已获得访问 API 并执行特定操作的授权（由`scope`已授予）。在我们之前使用的示例中，在您进行身份验证并同意待办事项应用程序可以对您的日历具有读/写访问权限后，a`access_token`从 Google 发送到待办事项应用程序。每次待办事项应用程序想要访问您的Google日历时，它都会向Google日历API发出请求，使用以下命令`access_token`在 HTTP 中`Authorization`页眉。

**注意：**客户端应将访问令牌视为不透明字符串。它们仅用于 API。您的客户端不应尝试对其进行解码或依赖于特定格式。`access_token` 

## 如何不使用每个令牌

现在我们已经看到了这些令牌的用途，让我们看看它们不能用于什么。

* `access_token`**不能用于身份验证。**它不包含有关用户的任何信息。它无法告诉我们用户是否已通过身份验证以及何时进行身份验证。
* `id_token`**不能用于 API 访问。**每个令牌都包含有关目标受众（收件人）的信息。根据 OpenID Connect 规范，每个请求的受众（声明）必须是发出身份验证请求的客户端。如果不是，则不应信任令牌。另一方面，API 需要一个将受众设置为 API 唯一标识符的令牌。因此，除非您同时控制客户端和 API，否则向 API 发送 将不起作用。此外，使用客户端已知的机密对 进行签名（因为它是颁发给特定客户端的）。这意味着，如果 API 接受此类令牌，则无法知道客户端是否修改了令牌（以添加更多作用域），然后再次对其进行签名。`audid_tokenclient_idid_tokenid_token`
## 比较代币

为了更好地理解我们刚刚阅读的内容，让我们看一下示例令牌的内容。

（解码的）内容`id_token`如下所示：

```plain
{
  "iss": "http://${account.namespace}/",
  "sub": "auth0|123456",
  "aud": "${account.clientId}",
  "exp": 1311281970,
  "iat": 1311280970,
  "name": "Jane Doe",
  "given_name": "Jane",
  "family_name": "Doe",
  "gender": "female",
  "birthdate": "0000-10-31",
  "email": "janedoe@example.com",
  "picture": "http://example.com/janedoe/me.jpg"
}
```
此令牌用于**向客户端验证用户身份**。请注意，令牌的受众（声明）设置为客户端的标识符，这意味着只有此特定客户端应使用此令牌。`aud`
为了进行比较，让我们看一下`access_token`:

```plain
{
  "iss": "https://${account.namespace}/",
  "sub": "auth0|123456",
  "aud": [
    "my-api-identifier",
    "https://${account.namespace}/userinfo"
  ],
  "azp": "${account.clientId}",
  "exp": 1489179954,
  "iat": 1489143954,
  "scope": "openid profile email address phone read:appointments email"
}
```
此令牌适用于**授权用户使用 API**.因此，它对客户端是完全不透明的，这意味着客户端不应该关心此令牌的内容，解码它或依赖于特定的令牌格式。请注意，令牌不包含有关用户本身的任何信息，除了其 ID （`sub`声明），它仅包含有关允许客户端在 API 上执行哪些操作的授权信息（`scope`索赔）。
由于在许多情况下，需要在API上检索其他用户信息，因此此令牌对于调用`/userinfo`API，它将返回用户的配置文件信息。因此，目标受众（`aud`声明） 的令牌是 API （`my-api-identifier`） 或`/userinfo`端点 （`https://${account.namespace}/userinfo`).

### 推荐资源进一步阅读

* [访问令牌](https://auth0.com/docs/tokens/access-token)
* [身份令牌](https://auth0.com/docs/tokens/id-token)
* [用于身份验证的 OAuth 问题](http://www.thread-safe.com/2012/01/problem-with-oauth-for-authentication.html)
* [OAuth 2.0 授权框架](https://tools.ietf.org/html/rfc6749)
* [OAuth 2.0 概述](https://auth0.com/docs/protocols/oauth2)
* [OpenID Connect 概述](https://auth0.com/docs/protocols/oidc)
## 使用 Auth0 保护应用程序

您是否正在构建一个[断续器](https://auth0.com/b2c-customer-identity-management),[断续器](https://auth0.com/b2b-enterprise-identity-management)或[断续器](https://auth0.com/b2e-identity-management-for-employees)工具？Auth0可以帮助您专注于对您最重要的事情，即产品的特殊功能。[身份验证0](https://auth0.com/)可以通过最先进的功能提高产品的安全性，例如[无密码](https://auth0.com/passwordless),[密码监控被破坏](https://auth0.com/breached-passwords)和[多因素身份验证](https://auth0.com/multifactor-authentication). 




