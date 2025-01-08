# Caching

Caching in **Orchard Core** is a fundamental concept that helps optimise performance by reducing redundant computations and database queries.

### IMemoryCache from .NET

Orchard Core utilises `IMemoryCache`, which is the caching abstraction provided by .NET Core with the library `Microsoft.Extensions.Caching.Memory`.

`IMemoryCache` is part of .NET Core and provides a straightforward in-memory caching mechanism for storing and retrieving data using keys. Orchard Core uses this internally for various purposes, such as caching content item data.

```csharp
//...
using Microsoft.Extensions.Caching.Memory;

//...
public class ProductController(
    IOrchardHelper orchard,
    //...
    IMemoryCache memoryCache) {

    //...
    
    [HttpGet("read-cache/{contentItemId}")]
    public async Task<IActionResult> ReadCacheDemo(string contentItemId)
    {
        ContentItem? productInformation;
    
        if (!memoryCache.TryGetValue(contentItemId, out productInformation))
        {
            productInformation = await orchard.GetContentItemByIdAsync(contentItemId);
            productInformation.DisplayText = $"Cached at {DateTime.Now:yyyy-MM-dd-HH:mm:ss}!";

            memoryCache.Set(contentItemId, productInformation);
        }

        return Ok(productInformation);
    }
}
```

The code above will return the product information of a given `contentItemId` with its display text changed to a text indicating when it is first accessed. Until we call the `memoryCache.Remove`, it should always show the same date time at the `DisplayText`.

### Signals

For invalidation, of course we can use the `Remove` function as shown below.

```csharp
memoryCache.Remove(contentItemId);
```

We can also make use of the Signal from Orchard Core to perform cache invalidation.

In Orchard Core, Signals are a powerful mechanism for invalidating cached content dynamically. We can raise the corresponding signal to invalidate the cache.

```csharp
//...
using Microsoft.Extensions.Caching.Memory;
using OrchardCore.Environment.Cache;

//...
public class ProductController(
    //...
    IMemoryCache memoryCache,
    ISignal orchardSignal) {

    private const string OrchardSignalKey = "OCBC.HeadlessCMS.Controllers.ProductController.OrchardSignalKey";
    
    //...
    
    [HttpGet("read-cache/{contentItemId}")]
    public async Task<IActionResult> ReadCacheDemo(string contentItemId)
    {
        ContentItem? productInformation;
        //...
        memoryCache.Set(contentItemId, productInformation, 
            orchardSignal.GetToken(OrchardSignalKey));
        //...
    }
    
    [HttpGet("invalidate-cache-with-signal/{contentItemId}")]
    public async Task<IActionResult> InvalidateCacheWithSignalDemo(string contentItemId)
    {
        // Raise a signal
        await orchardSignal.SignalTokenAsync(OrchardSignalKey);

        return Ok(new { Message = $"Raised signal to invalidate memory cache for {contentItemId}." });
    }
}
```

When the signal is raised, Orchard Core automatically invalidates all cached entries linked to that signal token.

### IDynamicCache

`IDynamicCache` is an abstraction provided by Orchard Core for **caching dynamic content** within the CMS.

To enabled this feature, please make sure the `OrchardCore.DynamicCache` feature is enabled.

<figure><img src="../.gitbook/assets/image (48).png" alt=""><figcaption><p>Dynamic Cache is enabled.</p></figcaption></figure>

While `IMemoryCache` is great for general-purpose object caching, `IDynamicCache` extends caching into Orchard Core-specific scenarios:

* **Dynamic Content**: Caching rendered HTML or partial views.
* **Signals**: Content-aware invalidation tied to updates in the Orchard CMS.
* **Tenant Awareness**: Built-in support for multitenancy.

```csharp
//...
using OrchardCore.DynamicCache;

//...
public class ProductController(
    //...
    IDynamicCache dynamicCache) {
    
    [HttpGet("read-dynamic-cache/{contentItemId}")]
    public async Task<IActionResult> ReadDynamicCacheDemo(string contentItemId)
    {
        ContentItem? productInformation;
        
        var cachedContentBytes = await dynamicCache.GetAsync(contentItemId);
        
        if (cachedContentBytes == null)
        {
            productInformation = await orchard.GetContentItemByIdAsync(contentItemId);
            productInformation.DisplayText = $"Dynamic Cached at {DateTime.Now:yyyy-MM-dd-HH:mm:ss}!";

            var serializedContentItem = JsonSerializer.Serialize(productInformation);

            await dynamicCache.SetAsync(contentItemId, Encoding.UTF8.GetBytes(serializedContentItem), 
                new DistributedCacheEntryOptions
                {
                    AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(1)
                }
            );
        } 
        else 
        {
            productInformation = JsonSerializer.Deserialize<ContentItem>(Encoding.UTF8.GetString(cachedContentBytes));
        }

        return Ok(productInformation);
    }
}
```

`IDynamicCache` doesn’t natively handle ETags or HTTP caching headers. We must manually compute and include ETag headers in our response.

### Response Caching from .NET

If we need traditional output caching, we can also implement it in our Orchard Core application by leveraging ASP.NET Core’s `ResponseCachingMiddleware`.

We need to add the relevant middleware to Program.cs of our core CMS project.

```csharp
//...

builder.Services.AddResponseCaching();

//...
app.UseResponseCaching();

//...
```

After that, we can have a controller method using Response Caching as follows.

```csharp
//...
public class ProductController(
    //...
    ) {

    [HttpGet("read-traditional-cache/{contentItemId}")]
    [ResponseCache(Duration = 60)]
    public async Task<IActionResult> ReadTraditionalCacheDemo(string contentItemId)
    {
        var productInformation = await orchard.GetContentItemByIdAsync(contentItemId);
        productInformation.DisplayText = $"Traditional Cached at {DateTime.Now:yyyy-MM-dd-HH:mm:ss}!";

        Response.Headers["ETag"] = Guid.NewGuid().ToString();
        return Ok(productInformation);
    }
}
```

