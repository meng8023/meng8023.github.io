---
title: IdentityServer4
tags: IdentityServer4
key: 20190823
---

IdentityServer4
===============

第一步  创建IdentityServer4项目(创建API项目)
------
1、创建一个 ASP.NET Core WEB API 项目
	选择不使用HTTPS
2、通过NuGet 添加 IdentityServer4 包（这里添加最新稳定版2.5.2）

3、创建Config.cs 类 编写 IdentityServer 方法

4、定义API资源 API是您要保护的系统中的资源

		/// <summary>
        /// 定义API资源 API是您要保护的系统中的资源
        /// </summary>
        /// <returns></returns>
        public static IEnumerable<ApiResource> GetApis()
        {
            return new List<ApiResource>
            {
                new ApiResource("api1", "My API")
            };
        }
		
5、定义客户端 

		/// <summary>
        /// 定义客户端
        /// </summary>
        /// <returns></returns>
        public static IEnumerable<Client> GetClients()
        {
            return new List<Client>
            {
                new Client
                {
                    ClientId = "client",

                    // no interactive user, use the clientid/secret for authentication
                    AllowedGrantTypes = GrantTypes.ClientCredentials,

                    // secret for authentication
                    ClientSecrets =
                    {
                        new Secret("secret".Sha256())
                    },

                    // scopes that client has access to
                    AllowedScopes = { "api1" }
                }
            };
        }
		
	ClientId 客户端ID
	AllowedGrantTypes 运行授权类型
	ClientSecrets 客户端密码（同时指定了加密方式）
	AllowedScopes 授权访问，对于API资源中定义的资源
	
6、内存中的标识资源

		/// <summary>
        /// 内存中的标识资源
        /// </summary>
        /// <returns></returns>
        public static IEnumerable<IdentityResource> GetIdentityResources()
        {
            return new List<IdentityResource>();
        }
	
7、配置IdentityServer

加载资源和客户端定义发生在Startup.cs中

	public void ConfigureServices(IServiceCollection services)
	{
		var builder = services.AddIdentityServer()
			.AddDeveloperSigningCredential()
			.AddInMemoryIdentityResources(Config.GetIdentityResources())
			.AddInMemoryApiResources(Config.GetApis())
			.AddInMemoryClients(Config.GetClients());

		// omitted for brevity
	}
	
8、将IdentityServer添加到管道中  app.UseIdentityServer();

		public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            app.UseIdentityServer();
            app.UseMvc();
        }
		
完成以上配置后访问 http://localhost:50242/.well-known/openid-configuration
可获得以下内容
您应该会看到所谓的发现文档。发现文档是身份服务器中的标准端点。客户端和API将使用发现文档来下载必要的配置数据。


	{
    "issuer": "http://localhost:50242",
    "authorization_endpoint": "http://localhost:50242/connect/authorize",
    "token_endpoint": "http://localhost:50242/connect/token",
    "userinfo_endpoint": "http://localhost:50242/connect/userinfo",
    "end_session_endpoint": "http://localhost:50242/connect/endsession",
    "check_session_iframe": "http://localhost:50242/connect/checksession",
    "revocation_endpoint": "http://localhost:50242/connect/revocation",
    "introspection_endpoint": "http://localhost:50242/connect/introspect",
    "device_authorization_endpoint": "http://localhost:50242/connect/deviceauthorization",
    "frontchannel_logout_supported": true,
    "frontchannel_logout_session_supported": true,
    "backchannel_logout_supported": true,
    "backchannel_logout_session_supported": true,
    "scopes_supported": [
        "api1",
        "offline_access"
    ],
    "claims_supported": [],
    "grant_types_supported": [
        "authorization_code",
        "client_credentials",
        "refresh_token",
        "implicit",
        "urn:ietf:params:oauth:grant-type:device_code"
    ],
    "response_types_supported": [
        "code",
        "token",
        "id_token",
        "id_token token",
        "code id_token",
        "code token",
        "code id_token token"
    ],
    "response_modes_supported": [
        "form_post",
        "query",
        "fragment"
    ],
    "token_endpoint_auth_methods_supported": [
        "client_secret_basic",
        "client_secret_post"
    ],
    "subject_types_supported": [
        "public"
    ],
    "id_token_signing_alg_values_supported": [
        "RS256"
    ],
    "code_challenge_methods_supported": [
        "plain",
        "S256"
    ],
    "request_parameter_supported": true
}


第二步， 创建业务API项目
1、创建一个 ASP.NET Core WEB API 项目
	选择不使用HTTPS
2、创建控制器 
这里我们创建一个名字是IdentityController的控制器
添加一个 Get 方法

		[HttpGet]
        public IActionResult Get()
        {
            return new JsonResult(from c in User.Claims select new { c.Type, c.Value });
        }
在控制器类中上添加 [Authorize] 类注释，开启授权

3、配置
最后一步是将身份验证服务添加到DI（依赖注入）和身份验证中间件到管道

验证传入令牌以确保它来自受信任的颁发者
验证令牌是否有效用于此API（也称为观众）
将Startup更新为如下所示：

	public class Startup
	{
		public void ConfigureServices(IServiceCollection services)
		{
			services.AddMvcCore()
				.AddAuthorization()
				.AddJsonFormatters();

			services.AddAuthentication("Bearer")
				.AddJwtBearer("Bearer", options =>
				{
					options.Authority = "http://localhost:50242";
					options.RequireHttpsMetadata = false;

					options.Audience = "api1";
				});
		}

		public void Configure(IApplicationBuilder app)
		{
			app.UseAuthentication();

			app.UseMvc();
		}
	}


AddAuthentication将身份验证服务添加到DI并配置"Bearer"为默认方案。
 UseAuthentication将身份验证中间件添加到管道中，以便在每次调用主机时自动执行身份验证。

http://localhost:5001/identity在浏览器上导航到控制器应返回401状态代码。
这意味着您的API需要凭证，现在受IdentityServer保护

第三步，客户端调用