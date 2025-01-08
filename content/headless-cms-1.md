# Managing API

Even though users can manage content items via the Orchard Core Administration UI, we can also build our own custom API endpoints to allow programatically managing the content items. This is particularly useful when you need custom logic, automation, or third-party integrations.

For content management, in order to handle content items programatically, we need to use another Orchard Core service, `IContentManager`. It provides an abstraction layer for managing the lifecycle of content items. Essentially, it is responsible for the CRUD operations of content items within Orchard Core.

### Versionable Content Item

The `Versionable` option is a content item option in Orchard Core that determines whether a content item should maintain multiple versions (such as drafts, published, and historical versions).&#x20;

When enabled, it unlocks features like:

* Draft and published separation.
* Version history for content.
* The ability to revert to previous versions.

### Content Management Behaviour with Versionable Disabled

Without the `Versionable` setting, Orchard Core does not maintain separate versions for draft and published states.

Editing a published content item will directly modify the live version. There will be no new draft created.

If `Versionable` is not enabled, the content item will only have **one single version**, meaning:

* Edits overwrite the current version directly.
* There’s no draft-published distinction or version history.

The `Latest` and `Published` flags both remain `true` when the content item is updated, as there is only one version in existence.

We can still unpublish and republish a content item, but the system won’t retain separate states or versions between these actions. Programmatic operations like creating a new draft, discarding drafts, or managing historical versions become irrelevant since no versioning exists.

In this page, we will thus only discuss content management behaviour with versionable **enabled**.

### Versionable Content Item Workflow

There will be several content item states as listed below.

<table><thead><tr><th width="157">State</th><th>Description</th><th data-hidden></th></tr></thead><tbody><tr><td>Draft</td><td>A version of the content item that is being worked on but is not yet visible to the public.<br>Exists only when modifications are made to a previously published item or when a new item is created but not published.</td><td></td></tr><tr><td>Published</td><td>The version of the content item that is publicly visible and available to end users. <br>There can only be one published version of a content item at any given time.</td><td></td></tr><tr><td>Historical</td><td>Older versions of content items that are retained for history or rollback purposes. <br>These versions are neither draft nor published but serve as a reference.</td><td></td></tr><tr><td>Removed</td><td>Content items can be "soft deleted" using methods like <code>RemoveAsync()</code>. This makes them invisible but does not delete them permanently.</td><td></td></tr></tbody></table>

Orchard Core uses a combination of the `Published` and `Latest` flags to implicitly determine the state of a content item. Here's how it works:

<table><thead><tr><th width="145">Latest</th><th width="155">Published</th><th>State</th></tr></thead><tbody><tr><td>True</td><td>True</td><td>Published</td></tr><tr><td>True</td><td>False</td><td>Draft</td></tr><tr><td>False</td><td>True</td><td>Historical</td></tr><tr><td>False</td><td>False</td><td>Removed</td></tr></tbody></table>

When a new content item is created, it starts as a draft. When a content item is published, its `Published` flag is set to `true`, and it becomes publicly visible. If we edit a published content item, Orchard Core creates a **new draft version**. The published version remains live until the draft is published.

### Create Content Item

To create a new content item for Product Information, we need to assign an instance of `ProductInformationPart` to the `Content` of the `contentItem`. For simplicity of the demo, we will use GET method to create a sample product information as shown below.

```csharp
using Microsoft.AspNetCore.Mvc;
using OCBC.ProductModule.Models;
using OrchardCore.ContentManagement;

namespace OCBC.HeadlessCMS.Controllers;

[ApiController]
[Route("api/v1/product-management")]
public class ProductManagementController(IContentManager contentManager) : Controller
{
    [HttpGet("create-sample")]
    public async Task<IActionResult> CreateSampleProductInformation()
    {
        var contentItem = await contentManager.NewAsync("ProductInformation");

        if (contentItem == null)
        {
            return BadRequest(new { Error = "Invalid content type." });
        }

        contentItem.DisplayText = $"New Product Information Created at {DateTime.Now:yyyy-MM-dd HH:mm:ss}";
        contentItem.Content.ProductInformationPart = new ProductInformationPart
        {
            ProductName = "Sample",
            Description = $"This is created at {DateTime.Now:yyyy-MM-dd HH:mm:ss}.",
        };

        await contentManager.CreateAsync(contentItem, VersionOptions.Draft);
            
        return Ok();
    }
}
```

Now, when we visit the Orchard Core Administration UI, we should be able to see the entry with ProductName and Description filled.

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption><p>This entry is programatically created.</p></figcaption></figure>

### Update Content Item

Before we can update a content item, we need to be able to fetch it. To retrieve a content item, we can use the `IContentManager.GetAsync()` method. The method accepts a content item ID and a VersionOptions parameter to specify which version of the content item to retrieve:

* **VersionOptions.Published:** Retrieves the latest published version.
* **VersionOptions.Latest:** Retrieves the most recent version (draft or published).
* **VersionOptions.Draft:** Retrieves the draft version only.
* **VersionOptions.DraftRequire:** Ensures that the draft version is retrieved (or created if it doesn’t exist).

As we discuss earlier, when we edit a published content item, Orchard Core creates a new draft version. So, we need to use `VersionOptions.DraftRequire`.

```csharp
[HttpGet("update-sample-with-version/{contentItemId}")]
public async Task<IActionResult> UpdateSampleProductInformationWithVersion(string contentItemId)
{
    // Retrieve the content item with a draft version (creates a draft if none exists)
    var contentItem = await contentManager.GetAsync(contentItemId, VersionOptions.DraftRequired);

    // Modify the content item
    contentItem.DisplayText += " - Updated XX";
    contentItem.Content.ProductInformationPart.ProductName += " - Updated";
    contentItem.Content.ProductInformationPart.Description += $" This is updated at {DateTime.Now:yyyy-MM-dd HH:mm:ss}.";

    // Save the changes (new draft version is saved)
    await contentManager.UpdateAsync(contentItem);

    // (Optional) Publish the new version
    await contentManager.PublishAsync(contentItem);

    return Ok(new { DateTime.Now });
}
```

Calling the method above will generate the following audit trail.

<figure><img src="../.gitbook/assets/image (46).png" alt=""><figcaption><p>New version is updated each time of updating and publishing our content item.</p></figcaption></figure>

If your `Versionable` is disabled for the content item, you can use the following simpler way of updating. The following code shows how we can update the content of an existing content item after retrieving it with its `contentItemId`.

```csharp
[HttpGet("update-sample/{contentItemId}")]
public async Task<IActionResult> UpdateSampleProductInformation(string contentItemId)
{
    var contentItem = await contentManager.GetAsync(contentItemId, VersionOptions.Latest);

    if (contentItem == null)
    {
        return BadRequest(new { Error = "Invalid content item id." });
    }

    contentItem.DisplayText += $" - Updated";
    contentItem.Content.ProductInformationPart.ProductName += " - Updated";
    contentItem.Content.ProductInformationPart.Description += $" This is updated at {DateTime.Now:yyyy-MM-dd HH:mm:ss}.";
        
    await contentManager.UpdateAsync(contentItem);
            
    return Ok();
}
```

### Publish/Unpublish Content Item

In Orchard Core, we can use `PublishAsync` and `UnpublishAsync` to manage the publishing state of a content item programmatically. Before that, we need to determine whether a content item has a published version using the `HasPublishedVersionAsync` method.

Unpublishing does not delete the published version but marks it as unavailable.

The following code shows how we can publish or unpublish a content item depends on its publishing state.

```
[HttpGet("publish-unpublish-sample/{contentItemId}")]
public async Task<IActionResult> PublishUnpublishSampleProductInformation(string contentItemId)
{
    var contentItem = await contentManager.GetAsync(contentItemId, VersionOptions.Latest);

    if (contentItem == null)
    {
        return BadRequest(new { Error = "Invalid content item id." });
    }

    if (await contentManager.HasPublishedVersionAsync(contentItem))
    {
        await contentManager.UnpublishAsync(contentItem);
    } 
    else
    {
        await contentManager.PublishAsync(contentItem);
    }
            
    return Ok();
}
```

### Delete Content Item

There is a method called `RemoveAsync` we can use to delete content item. However, we need to take note that it only removes `ContentItem.Latest` and `ContentItem.Published` flags from a content item, making it invisible from the system. It does **not physically delete** the content item.

```csharp
await contentManager.RemoveAsync(contentItem);
```

There is also another method called `DiscardDraftAsync` which deletes the draft version of a content item. Take note that it only removes the draft version of a content item if it exists, leaving the **published version (if any) untouched**.

```csharp
await contentManager.DiscardDraftAsync(contentItem);
```

So what will happen if we perform `UnpublishAsync` or `RemoveAsync` on a content item and then call `DiscardDraftAsync` ? Here is what happens step by step:

Since we know that when we unpublish a content item using `UnpublishAsync`, the `Published` flag is removed from the content item. The content item will still have the `Latest` flag set, indicating that it's in a draft state. So, calling `DiscardDraftAsync` after using `UnpublishAsync` should have the same effect as `RemoveAsync`. Nevertheless , the content item is still stored in the database.

Let's go through some examples. For example, we have a published content item which can be represented as the following JSON.

```json
{
    "ContentItemId": "4g32v6xabcemb06eq9wazyekwa",
    "ContentItemVersionId": "4fa49x0ntk5wb65n387pb1wwcf",
    "ContentType": "ProductInformation",
    "DisplayText": "New Product Information Created at 2024-12-13 21:07:07",
    "Latest": true,
    "Published": true,
    "ModifiedUtc": "2024-12-13T13:07:07.621872Z",
    "PublishedUtc": "2024-12-13T13:07:27.600396Z",
    "CreatedUtc": "2024-12-13T13:07:07.621872Z",
    "Owner": "4gyfgah3k3ness5adpmffgntnc",
    "Author": "admin",
    "CommonPart": {

    },
    "ProductInformationPart": {
        ...
    }
}
```

Now, if we call `RemoveAsync` on it, both the `Latest` and `Published` flags will be set to false, so the end result of calling `RemoveAsync` should be as follows.

```json
{
    "ContentItemId": "4g32v6xabcemb06eq9wazyekwa",
    "ContentItemVersionId": "4fa49x0ntk5wb65n387pb1wwcf",
    "ContentType": "ProductInformation",
    "DisplayText": "New Product Information Created at 2024-12-13 21:07:07",
    "Latest": false,
    "Published": false,
    ...
}
```

At this moment, this content item is still queryable with the following code.

```csharp
await orchard.QueryContentItemsAsync(q => 
            q.Where(c => c.ContentType == "ProductInformation"));
```

However, it is not queryable with the following because the `Latest` flag is already set to false.

```csharp
await contentManager.GetAsync(contentItemId, VersionOptions.Latest);
```

You may wonder, what if we use `DiscardDraft` on a published content item? The answer is simple. As shown in the following screenshot, there will be an error thrown.

<figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption><p>Orchard Core does not allow us to use DiscardDraft one a content item which is not a draft.</p></figcaption></figure>

Now, let's move on to a different content item which is a draft and is represented as the following JSON.

```
{
    "ContentItemId": "4mjy7gk6tcwmfyrjere6bt6k49",
    "ContentItemVersionId": "4nxd6rz5daeh6xj91yxrb8w05g",
    "ContentType": "ProductInformation",
    "DisplayText": "New Product Information Created at 2024-12-13 21:21:07",
    "Latest": true,
    "Published": false,
    ...
}
```

Will using `RemoveAsync` on a draft throws an exception also? Nope, because what `RemoveAsync` will do is simply set the `Latest` flag of a draft to false since its `Published` flag is already false.

In fact, calling `DiscardDraft` on a draft will do the same as calling `RemoveAsync` on it because the outcome is always having both `Latest` and `Published` flag set to false.

```
{
    "ContentItemId": "4mjy7gk6tcwmfyrjere6bt6k49",
    "ContentItemVersionId": "4nxd6rz5daeh6xj91yxrb8w05g",
    "ContentType": "ProductInformation",
    "DisplayText": "New Product Information Created at 2024-12-13 21:21:07",
    "Latest": false,
    "Published": false,
    ...
}
```

And yes, for a discarded draft, it is still queryable with `QueryContentItemsAsync` but not `GetAsync`, as we discussed above.
