---
title: IdentityServer4 实现 OpenID Connect 和 OAuth 2.0
tags: IdentityServer4 OpenID OAuth
key: 20190828
---

IdentityServer4 实现 OpenID Connect 和 OAuth 2.0
===============

转<https://www.cnblogs.com/xishuai/p/identityserver4-implement-openid-connect-and-oauth2.html>

关于 OAuth 2.0 的相关内容，点击查看：ASP.NET WebApi OWIN 实现 OAuth 2.0

	OpenID 是一个去中心化的网上身份认证系统。对于支持 OpenID 的网站，用户不需要记住像用户名和密码这样的传统验证标记。取而代之的是，他们只需要预先在一个作为 OpenID 身份提供者（identity provider, IdP）的网站上注册。OpenID 是去中心化的，任何网站都可以使用 OpenID 来作为用户登录的一种方式，任何网站也都可以作为 OpenID 身份提供者。OpenID 既解决了问题而又不需要依赖于中心性的网站来确认数字身份。

OpenID 相关基本术语：

	最终用户（End User）：想要向某个网站表明身份的人。
	
	标识（Identifier）：最终用户用以标识其身份的 URL 或 XRI。
	
	身份提供者（Identity Provider, IdP）：提供 OpenID URL 或 XRI 注册和验证服务的服务提供者。
	
	依赖方（Relying Party, RP）：想要对最终用户的标识进行验证的网站。

以上概念来自：https://zh.wikipedia.org/wiki/OpenID

针对 .NET Core 跨平台，微软官方并没有针对 OAuth 2.0 的实现（Microsoft.AspNetCore.Authentication.OAuth组件，仅限客户端），IdentityServer4 实现了 ASP.NET Core 下的 OpenID Connect 和 OAuth 2.0，IdentityServer4 也是微软基金会成员。

阅读目录：

	OpenID 和 OAuth 的区别
	
	客户端模式（Client Credentials）
	
	密码模式（resource owner password credentials）
	
	简化模式-With OpenID（implicit grant type）
	
	简化模式-With OpenID & OAuth（JS 客户端调用）
	
	混合模式-With OpenID & OAuth（Hybrid Flow）
	
	ASP.NET Core Identity and Using EntityFramework Core for configuration data
	
	开源地址：https://github.com/yuezhongxin/IdentityServer4.Demo
	
1. OpenID 和 OAuth 的区别
-------------------------
简单概括：

	OpenID：authentication（认证），用户是谁？
	
	OAuth：authorization（授权），用户能做什么？

其实，OAuth 的密码授权模式和 OpenID 有些类似，但也不相同，比如用户登录落网选择微博快捷登录方式，大致的区别：

	OAuth：用户在微博授权页面输入微博的账号和密码，微博验证成功之后，返回 access_token，然后落网拿到 access_token 之后，再去请求微博的用户 API，微博授权中心验证 access_token，如果验证通过，则返回用户 API 的请求数据给落网。
	
	OpenID：落网可以没有用户的任何实现，落网需要确认一个 URL 标识（可以是多个），然后用户登录的时候，选择一个 URL 进行登录（比如微博），跳转到微博 OpenID 登录页面，用户输入微博的账号和密码，微博验证成功之后，按照用户的选择，返回用户的一些信息。

可以看到，OAuth 首先需要拿到一个授权（access_token），然后再通过这个授权，去资源服务器（具体的 API），获取想要的一些数据，上面示例中，用户 API 只是资源服务器的一种（可以是视频 API、文章 API 等等），在这个过程中，OAuth 最重要的就是获取授权（四种模式），获取到授权之后，你就可以通过这个授权，做授权范围之类的任何事了。

而对于 OpenID 来说，授权和它没任何关系，它只关心的是用户，比如落网，可以不进行用户的任何实现（具体体现就是数据库没有 User 表），然后使用支持 OpenID 的服务（比如微博），通过特定的 URL 标识（可以看作是 OpenID 标识），然后输入提供服务的账号和密码，返回具体的用户信息，对于落网来说，它关心的是用户信息，仅此而已。
