# Serving API

Orchard Core simplifies creating API endpoints by leveraging ASP.NET Core's MVC framework. We can define a controller with specific methods, and Orchard Core will automatically expose those as endpoints.

Let's start a simple controller in the CMS core project with the following code.

```csharp
using Microsoft.AspNetCore.Mvc;

namespace OCBC.HeadlessCMS.Controllers;

[ApiController]
[Route("api/v1/product")]
public class ProductController : Controller
{
    [HttpGet("product-info")]
    public IActionResult GetProductInformation()
    {
        return Ok(new { message = "Hello from Orchard Core!" });
    }
}
```

We should be seeing the message being printed when we visit the endpoint.

Since we are building this serving API, so we need a way to retrieve the content items from Orchard Core. In Orchard Core, we can use `IOrchardHelper` in to interact with content items and other Orchard Core services easily. `IOrchardHelper` provides utility methods to retrieve content items, execute queries, and work with shapes, making it incredibly useful for API development.

There are a few key methods in `IOrchardHelper`, as explained in the following table.

| Method                                                                                                                |                                                                           |
| --------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| GetContentItemByIdAsync(string contentItemId)                                                                         | Retrieve a specific content item by its ID.                               |
| QueryContentItemsAsync(Func\<IQuery, IQuery> query)                                                                   | Query content items using LINQ-like syntax.                               |
| ImageResizeUrl(string imagePath, int? width = null, int? height = null, ResizeMode resizeMode = ResizeMode.Undefined) | Returns a URL with custom resizing parameters for an existing image path. |
| SanitizeHtml(string html)                                                                                             | Sanitises an HTML string.                                                 |

{% hint style="info" %}
Learn more about `IOrchardHelper` methods [here](https://docs.orchardcore.net/en/main/reference/core/Razor/).
{% endhint %}

For example, to retrieve all the published product information with "ProductInformation" as the ContentType, we will need to make use of the `QueryContentItemsAsync`, as demonstrated below.

```csharp
using Microsoft.AspNetCore.Mvc;
using OrchardCore;

namespace OCBC.HeadlessCMS.Controllers;

[ApiController]
[Route("api/v1/product")]
public class ProductController(IOrchardHelper orchard) : Controller
{
    [HttpGet("product-info")]
    public async Task<IActionResult> GetProductInformation()
    {
        var productInformation = await orchard.QueryContentItemsAsync(q => 
            q.Where(c => c.ContentType == "ProductInformation"));
            
        return Ok(productInformation);
    }
}
```

With this, the output of the endpoint should be a JSON as follows.

```json
{
  "result": [
    {
      "ContentItemId": "4fdpjjm5cgqz3y5zzaz72ccp4a",
      "ContentItemVersionId": "4y396xfxkk7p039g1545662qne",
      "ContentType": "ProductInformation",
      "DisplayText": "FD03 - Durian",
      "Latest": true,
      "Published": true,
      "ModifiedUtc": "2024-12-11T09:25:13.896787Z",
      "PublishedUtc": "2024-12-11T09:25:13.918769Z",
      "CreatedUtc": "2024-12-11T09:25:13.896787Z",
      "Owner": "4gyfgah3k3ness5adpmffgntnc",
      "Author": "admin",
      "CommonPart": {

      },
      "ProductInformationPart": {
        "ProductImage1": {
          "Paths": [
            "1200x675_cmsv2_2e48fe04-b71f-585d-af85-4be91dd5a2d6-7869754.webp"
          ],
          "MediaTexts": [
            ""
          ]
        },
        "ProductImage2": {
          "Paths": [
            "great-wall-china-badaling-rain-28395675.webp"
          ],
          "MediaTexts": [
            ""
          ]
        },
        "ProductImage3": {
          "Paths": [
            "One Pillar Pagoda Hanoi.jpg"
          ],
          "MediaTexts": [
            ""
          ]
        },
        "ProductName": "Durian",
        "ChineseProductName": "榴莲",
        "Model": "FD03",
        "Description": "Durian from Malaysia."
      }
    },
    ...
  ],
  "id": 1,
  "exception": null,
  "status": 5,
  "isCanceled": false,
  "isCompleted": true,
  "isCompletedSuccessfully": true,
  "creationOptions": 0,
  "asyncState": null,
  "isFaulted": false
}
```

The list above will include both the published and draft content items. If we only want to show the published ones, we can do as follows.

```csharp
[HttpGet("product-info/published")]
public async Task<IActionResult> GetPublishedProductInformation()
{
    var productInformation = await orchard.QueryContentItemsAsync(q => 
        q.Where(c => c.ContentType == "ProductInformation" && c.Published));
            
    return Ok(productInformation);
}
```

If we have the `ContentItemId`, we can use it to retrieve the corresponding content item too.

```csharp
[HttpGet("product-info/{contentItemId}")]
public async Task<IActionResult> GetProductInformation(string contentItemId)
{
    var productInformation = await orchard.GetContentItemByIdAsync(contentItemId);
            
    return Ok(productInformation);
}
```

So, when we visit the endpoint `/api/v1/product/product-info/4fdpjjm5cgqz3y5zzaz72ccp4a`, we will receive the following JSON output.

```json
{
  "ContentItemId": "4fdpjjm5cgqz3y5zzaz72ccp4a",
  "ContentItemVersionId": "4y396xfxkk7p039g1545662qne",
  "ContentType": "ProductInformation",
  "DisplayText": "FD03 - Durian",
  "Latest": true,
  "Published": true,
  "ModifiedUtc": "2024-12-11T09:25:13.896787Z",
  "PublishedUtc": "2024-12-11T09:25:13.918769Z",
  "CreatedUtc": "2024-12-11T09:25:13.896787Z",
  "Owner": "4gyfgah3k3ness5adpmffgntnc",
  "Author": "admin",
  "CommonPart": {

  },
  "ProductInformationPart": {
    "ProductImage1": {
      "Paths": [
        "1200x675_cmsv2_2e48fe04-b71f-585d-af85-4be91dd5a2d6-7869754.webp"
      ],
      "MediaTexts": [
        ""
      ]
    },
    "ProductImage2": {
      "Paths": [
        "great-wall-china-badaling-rain-28395675.webp"
      ],
      "MediaTexts": [
        ""
      ]
    },
    "ProductImage3": {
      "Paths": [
        "One Pillar Pagoda Hanoi.jpg"
      ],
      "MediaTexts": [
        ""
      ]
    },
    "ProductName": "Durian",
    "ChineseProductName": "榴莲",
    "Model": "FD03",
    "Description": "Durian from Malaysia."
  }
}
```
