[合集 \- .NET Core(12\)](https://github.com)[1\.多个 .NET Core SDK 版本之间进行切换 global.json03\-12](https://github.com/chenyishi/p/18066796)[2\.HttpClientHandler VS SocketsHttpHandler03\-10](https://github.com/chenyishi/p/18064531)[3\..Net Core中使用DiagnosticSource进行日志记录03\-12](https://github.com/chenyishi/p/18068309)[4\.DiagnosticSource DiagnosticListener 无侵入式分布式跟踪03\-14](https://github.com/chenyishi/p/18071178)[5\.LoggerMessageAttribute 高性能的日志记录03\-15](https://github.com/chenyishi/p/18073599)[6\..Net Core 你必须知道的source\-generators03\-16](https://github.com/chenyishi/p/18073694):[FlowerCloud机场节点订阅](https://dahelaoshi.com)[7\..NET Core使用 CancellationToken 取消API请求03\-17](https://github.com/chenyishi/p/18075600)[8\.使用 LogProperties source generator 丰富日志03\-18](https://github.com/chenyishi/p/18078355)[9\..Net Core 使用 TagProvider 与 Enricher 丰富日志03\-19](https://github.com/chenyishi/p/18081945)[10\.C\# 12 拦截器 Interceptors03\-20](https://github.com/chenyishi/p/18082725)[11\.为什么推荐在 .NET 中使用 YAML 配置文件12\-23](https://github.com/chenyishi/p/18624234)12\..NET 9 中的 多级缓存 HybridCache12\-24收起
#### HybridCache是什么


在 .NET 9 中，Microsoft 将 HybridCache 带入了框架体系。


HybridCache 是一种新的缓存模型，设计用于封装本地缓存和分布式缓存，使用者无需担心选择缓存类型，从而优化性能和维护效率。


实际上，HybridCache 基于 IDistributedCache 提供的接口和操作，但增加了一些其他的特性，如封装两类不同缓存库（本地和分布），支持标签删除（Tag\-based Cache Eviction）和约束选项。


**需要注意**的是，HybridCache仍处于preview阶段。




---


#### HybridCache 与 IDistributedCache 的区别


**IDistributedCache：**


1. 仅支持分布式缓存，如 Redis、SQL Server、MemoryCache。
2. 选择依赖于目标缓存和管理设备。
3. 不支持标签删除，只能基于键值操作。


**HybridCache：**


1. 支持封装本地和分布式缓存，在读取时优先读取本地缓存，如本地不存在，再读取分布式。
2. 支持标签删除，通过指定标签管理缓存内容。
3. 选项更加精简，支持自动化操作和选项约束。



---


#### HybridCache 的好处


1. **性能优化：** 本地缓存速度超过分布式，使用 HybridCache 可以减少读取分布缓存库时的延迟。
2. **精简化工程：** 使用者不需再自行核实选择哪个缓存，增加了工程效率。
3. **标签管理：** 缓存标签记录不同类型数据，便于分类管理和删除缓存。
4. **安全性：** 支持选项约束，使缓存操作更严格，防止错误使用和内容亏失。



---


#### 代码示例


以下代码展示如何使用 HybridCache：


##### 1\. 添加缓存服务



```
var builder = WebApplication.CreateBuilder(args);

// 注册 HybridCache 服务
builder.Services.AddHybridCache();

// 注册 Redis 缓存服务，为 HybridCache 提供分布式缓存
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration.GetConnectionString("RedisConnectionString");
});

builder.Services.AddControllers();
```

##### 2\. 实现接口操作


**读取缓存**



```
[HttpGet("GetCache")]
public async Task<string[]> Get()
{
    return await _cache.GetOrCreateAsync(
        "a-1", async cancel => await Task.FromResult(Summaries)
    );
}

```

**删除缓存**



```
[HttpGet("DeleteCache")]
public async Task<bool> Delete()
{
    await _cache.RemoveAsync("a-1");
    return true;
}

```

**通过标签读取缓存**



```
[HttpGet("GetCacheByTag")]
public async Task<string[]> GetCacheByTag()
{
    var tags = new List<string> { "tag1", "tag2", "tag3" };
    var entryOptions = new HybridCacheEntryOptions
    {
        Expiration = TimeSpan.FromMinutes(1),
        LocalCacheExpiration = TimeSpan.FromMinutes(1)
    };
    return await _cache.GetOrCreateAsync(
        "a-1", async cancel => await Task.FromResult(Summaries),
        entryOptions, tags
    );
}

```

**通过标签删除缓存**



```
[HttpGet("DeleteCacheByTag")]
public async Task<bool> DeleteCacheByTag()
{
    var tags = new List<string> { "tag1" };
    await _cache.RemoveByTagAsync(tags);
    return true;
}

```



---


#### 小结


.NET 9 的 HybridCache 提供了一种便捷且高效的缓存解决方案，将本地缓存和分布式缓存无缝结合，为开发者简化了缓存逻辑，同时提供了更多高级功能，如标签管理和选项约束。通过代码示例可以看出，HybridCache 的操作直观且易于实现，非常适合现代应用场景。


如果你正在使用 .NET 9，尝试将 HybridCache 应用于你的项目中，体验其高效与简洁！


![](https://images.cnblogs.com/cnblogs_com/chenyishi/1348350/o_240408130234_wx.png)
