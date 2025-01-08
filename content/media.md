# Media

In Orchard Core, media, such as images, videos, and files, are managed in the **Media Library**, a module that serves as a **central storage and management area for all our media files**. Once a file is in the Media Library, it can be easily used across our CMS without needing to upload it again.

### Supported File Types

Media Library supports a wide range of file types. We can get a list of default allowed file extensions and other media options by visiting Configuration > Media > Media Options.

<figure><img src="../.gitbook/assets/image (20).png" alt=""><figcaption><p>Read more about the Media Options on the documentation.</p></figcaption></figure>

Even though the message in Orchard Core Admin UI says "Media is configured with appsettings.json", there is no effect if we directly update our `appsettings.json`. Instead, to configure the Media Options, we need to apply the following in our core CMS project.

```csharp
using OrchardCore.Media;

var builder = WebApplication.CreateBuilder(args);

builder.Services
    .AddOrchardCms()
    // Orchard Specific Pipeline
    .ConfigureServices(services => {
        services.PostConfigure<MediaOptions>(o => o.AllowedFileExtensions = [
            // Images
            ".jpg",
            ".jpeg",
            ".png",
            ".webp",
            ".heic",

            // Documents
            ".pdf",
            ".xls",
            ".xlsx",

            // Other
            ".json",
            ".zip",
        ]);
    })
    .Configure( (app, routes, services) => {
        // ...
    });
```

If our configuration is correct, the Media Options page will be updated with the new list of supported file extensions according to our configuration. Now, whenever we upload a file with unsupported file extension, there will be error displayed, as shown in the following screenshot.

<figure><img src="../.gitbook/assets/image (21).png" alt=""><figcaption><p>A message box telling users they have uploaded file with unsupported file extension.</p></figcaption></figure>

The **MediaField** is designed to work with the **Media Library** module.



