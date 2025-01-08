# Localisation with Portable Object

For localisation in Orchard Core, we need to make sure that the module `OrchardCore.Localization` is enabled. The module provides the infrastructure necessary to support the Portable Object (PO) localisation file format.

### Setup

It is suggested to put your localization files in the `/Localization/` folder.

The PO files need to be included in the publish output directory. Hence, we need to add the following configurations to our `.csproj` file to include them as Content.

```xml
<ItemGroup>
    <Folder Include="Localization\" />
    <Content Include="Localization\**">
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </Content>
</ItemGroup>
```

### Example of PO File

```
#: Weather summary (Controller)
msgctxt "OCBC.HeadlessCMS.Controllers.ProductController"
msgid "weather_Freezing"
msgstr "寒冷"

msgctxt "OCBC.HeadlessCMS.Controllers.ProductController"
msgid "weather_Cool"
msgstr "清爽"

msgctxt "OCBC.HeadlessCMS.Controllers.ProductController"
msgid "weather_Mild"
msgstr "温和"

msgctxt "OCBC.HeadlessCMS.Controllers.ProductController"
msgid "weather_Warm"
msgstr "暖和"

msgctxt "OCBC.HeadlessCMS.Controllers.ProductController"
msgid "weather_Hot"
msgstr "炎热"

msgctxt "OCBC.HeadlessCMS.Controllers.ProductController"
msgid "weather_Scorching"
msgstr "灼热"
```

When working with Orchard Core's `IStringLocalizer` implementation, the `msgctxt` (context string) in the PO file must match the full name of the .NET type we are using with `IStringLocalizer<T>`. This is because Orchard Core uses the `msgctxt` to distinguish between translations for different types, even if the `msgid` values are the same.

### Inject IStringLocalizer in Controller

Now, let's see how we can retrieve translated text from PO files in the controller.

For demo purpose, we will create the following endpoint in our Service API to return the Chinese translated text for `weather_Hot`.

```
[HttpGet("po")]
public async Task<IActionResult> GetPortableObjectDemoInformation()
{
    // Hardcode the locale for testing
    var testCulture = "zh-CN";
    CultureInfo.CurrentCulture = new CultureInfo(testCulture);
    CultureInfo.CurrentUICulture = new CultureInfo(testCulture);

    // Use the localizer to get a string
    var localizedString = stringLocalizer["weather_Hot"];

    // Return the localized string
    return Ok(new { Locale = testCulture, Translation = localizedString });
}
```

When visiting the endpoint above, we should get the correct translated text in Chinese, as shown in the following screenshot.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption><p>The correct Chinese text is retrieved.</p></figcaption></figure>

### Inject IStringLocalizer in Service

If we are going to inject IStringLocalizer in our services in order to get the translated texts, then we need to prepare another set of PO entries with the proper `msgctxt`.

Let's take an example of injecting IStringLocalizer to the following `DemoService`.

```
using System.Globalization;
using Microsoft.Extensions.Localization;

namespace OCBC.HeadlessCMS.Services;

public class DemoService(IStringLocalizer<DemoService> stringLocalizer) : IDemoService
{
    public object GetDemoTranslatedText()
    {
        // Hardcode the locale for testing
        var testCulture = "zh-CN";
        CultureInfo.CurrentCulture = new CultureInfo(testCulture);
        CultureInfo.CurrentUICulture = new CultureInfo(testCulture);

        // Use the localizer to get a string
        var localizedString = stringLocalizer["weather_Hot"];

        // Return the localized string
        return new { Locale = testCulture, Translation = localizedString };
    }
}
```

Even though we are still getting the same `weather_Hot` for the same locale as how we did in the controller earlier, we still need to add in the following content to our PO file above.

```
#: Weather summary (Service)
msgctxt "OCBC.HeadlessCMS.Services.DemoService"
msgid "weather_Freezing"
msgstr "寒冷"

msgctxt "OCBC.HeadlessCMS.Services.DemoService"
msgid "weather_Cool"
msgstr "清爽"

msgctxt "OCBC.HeadlessCMS.Services.DemoService"
msgid "weather_Mild"
msgstr "温和"

msgctxt "OCBC.HeadlessCMS.Services.DemoService"
msgid "weather_Warm"
msgstr "暖和"

msgctxt "OCBC.HeadlessCMS.Services.DemoService"
msgid "weather_Hot"
msgstr "炎热"

msgctxt "OCBC.HeadlessCMS.Services.DemoService"
msgid "weather_Scorching"
msgstr "灼热"
```

### About Supported Cultures

Orchard Core application can work without explicitly configuring `SupportedCultures`, but there are important caveats and limitations to consider.

Even though Orchard Core can still load translations from PO files if their filenames match the requested culture (e.g., en.po for French or zh-CN.po for Simplified Chinese), there is no guarantee of consistent behaviour if unsupported or unexpected cultures are requested.

For consistent behaviour and better performance, always configure `DefaultRequestCulture`, `SupportedCultures` and `SupportedUICultures`. in `Program.cs` of the CMS projec&#x74;**.** This ensures:

* Clear boundaries for the app's localisation support;
* Reliable and predictable fallback mechanisms;
* Avoidance of unexpected behaviours with unsupported or incorrectly specified cultures.

```csharp
builder.Services.Configure<RequestLocalizationOptions>(options =>
{
    var supportedCultures = new[] { 
        new CultureInfo("en-GB"), 
        new CultureInfo("zh-CN") 
    };

    options.DefaultRequestCulture = new RequestCulture("en-GB");
    options.SupportedCultures = supportedCultures;
    options.SupportedUICultures = supportedCultures;
});
```
