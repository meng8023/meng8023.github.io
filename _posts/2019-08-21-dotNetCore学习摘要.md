---
title: 文章标题
tags: TeXt
key: 20181112
---

内容标题
===============

Price 属性会强制执行最小值和最大值

{% highlight ruby linenos %}

using System.ComponentModel.DataAnnotations;

namespace ContosoPets.Api.Models
{
    public class Product
    {
        public long Id { get; set; }

        [Required]
        public string Name { get; set; }

        [Required]
        [Range(minimum: 0.01, maximum: (double) decimal.MaxValue)]
        public decimal Price { get; set; }
    }
}

{% endhighlight ruby %}



前面的操作中使用的每个 ActionResult 都映射到下表中对应的 HTTP 状态代码。
ASP.NET Core
操作结果 HTTP 状态代码 说明
NoContent 204 该产品已在数据库中更新。
BadRequest 400 请求正文的 Id 值与路由的 id 值不匹配。
BadRequest 为隐式 400 请求正文的 Product 对象无效。

{% highlight ruby linenos %}
[HttpPut("{id}")]
	public async Task<IActionResult> Update(long id, Product product)
	{
		if (id != product.Id)
		{
			return BadRequest();
		}

		_context.Entry(product).State = EntityState.Modified;
		await _context.SaveChangesAsync();

		return NoContent();
	}
{% endhighlight ruby %}



向 Web API 发送无效的 HTTP POST 请求：

{% highlight ruby linenos %}
curl -i -k \
    -H "Content-Type: application/json" \
    -d "{\"name\":\"Plush Squirrel\",\"price\":0.00}" \
    https://localhost:5001/api/Products
	
{% endhighlight ruby %}

在上述命令中：
-i 显示 HTTP 响应标头。
-d 表示 HTTP POST 操作，并定义请求正文。
-H 指示请求正文采用 JSON 格式。 标头的值将替代默认内容类型 application/x-www-form-urlencoded。
该命令返回 HTTP 400 状态代码，因为控制器的 [ApiController] 属性会在请求正文上触发模型验证。 MVC 的模型绑定器尝试将请求的 -d JSON 转换为 Product 对象。 模型验证失败，因为请求的 Price 值小于最小值 0.01。