---
title: WCF Restful 的实现
tags: WCF
---

首先这里提供一个微软官方示例下载的地址
<https://download.microsoft.com/download/1/5/9/159D6D71-7728-45D4-BC15-5DF1F2DDCD94/WF_WCF_Samples.exe>
这是一个压缩包文件，下载后执行解压就可以了
你可以在解压后的 这个WF_WCF_Samples\WCF\Extensibility\Web\FormPost\CS  路径找到 示例项目。

Restful 实现
------------

让请求区分 POST、DELETE、PUT 等不同的请求
使用协议 webHTTP
方法标注如下
{% highlight ruby linenos %}
        [WebInvoke(Method = "DELETE", UriTemplate = "{id}"), Description("Deletes the specified customer from customers collection. Returns NotFound if there is no such customer.")]
        public void DeleteCustomer(string id)
        {
            if (!customers.ContainsKey(id))
            {
                throw new WebFaultException(HttpStatusCode.NotFound);
            }
            else
            {
                lock (writeLock)
                {
                    customers.Remove(id);
                }
            }
        }
{% endhighlight ruby %}

需要注意的是，如果我们是创建使用工具直接创建一个支持Ajax请求的服务类，
方法注解是这样子得
{% highlight ruby linenos %}
[OperationContract]
        public void DoWork()
        {
            // 在此处添加操作实现
            return;
        }
{% endhighlight ruby %}

我们要实现Restful

### 第一步 ###

	添加 WebInvoke 或 WebGet 注解
  
### 第二步 ###

	也是很容易忽略的异步 要删除 [OperationContract] 的注解。
	
	
请求帮助页面的开启
-----------------
在配置文件的

{% highlight ruby linenos %}
<system.serviceModel>
  <behaviors>
    <endpointBehaviors>
      <endpointBehaviors> 的 <behavior>
        <webHttp>
{% endhighlight ruby %}
添加 helpEnabled=true 开启
{% highlight ruby linenos %}
<webHttp helpEnabled="true"/>
{% endhighlight ruby %}

开启后你就可以使用你的服务请求地址+'/help' 查看自动生成的帮助页面了，在方法注解中的Description属性的值就是方法的说明。

http://localhost:8000/Customers/help
