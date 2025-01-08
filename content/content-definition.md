# Content Definition

`Content Definitions` are a record of the `Content Types`, `Content Parts`, and `Content Fields`.

{% stepper %}
{% step %}
### Content Type

A blueprint for creating items in our CMS. A Content Type tells Orchard Core what kind of information (fields and parts) is included for items of that type. For example, if we want to manage products in an online store, we will create a Content Type called `Product`.
{% endstep %}

{% step %}
### Content Part

A component that can be added to many Content Types. For example, we can have a `TitlePart` provides a title for any type of content type.
{% endstep %}

{% step %}
### Content Field

A smaller unit that holds specific types of data, such as text, numbers, or dates. If "Product Name" is a short answer, then `TextField` is the box where we type that short answer.
{% endstep %}
{% endstepper %}

When the `File Content Definition` feature is enabled it stores content definitions in a `ContentDefinition.json` file.

In the subsequent sections below, we will go through the steps of programmatically creating a Content Type for Product Information in our CMS.

### Step 1: Create a Module for Product Information

Firstly, we need to create a new module where we will write the code for the Product Information.

```bash
dotnet new ocmodulecms -n OCBC.ProductModule
```

Please do not place module project in the core CMS project. Instead, make sure the two projects are sibling to each other, as shown below. Otherwise, running `dotnet build` will have issues.

```
MyCMSProject/
├── OCBC.HeadlessCMS/
    ├── ...
    ├── appsettings.json
    ├── appsettings.Development.json
    ├── OCBC.HeadlessCMS.csproj
    ├── Program.cs
├── OCBC.ProductModule/
    ├── ...
    ├── OCBC.ProductModule.csproj
    ├── Startup.cs
├── .gitignore
├── MyCMSProject.sln
```

Secondly, add the newly created module project to the solution.

```
dotnet sln add OCBC.ProductModule/OCBC.ProductModule.csproj
```

Thirdly, add a project reference to our new module in the CMS host application, i.e. `OCBC.HeadlessCMS.csproj`:

```xml
<ItemGroup>
  <ProjectReference Include="..\OCBC.ProductModule\OCBC.ProductModule.csproj" />
</ItemGroup>
```

### Step 2: Enable the New Module

We can now verify if we have setup our module correctly by starting our CMS application. Here we simply need to do `dotnet run` at the CMS project.

Please navigate to the **Admin Dashboard** > **Configuration** > **Features**, as shown in the screenshot below. We should be able to locate the module (`OCBC.ProductModule`) in the list of features. We can proceed to enable it, if this module is created after the CMS has been setup.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Locate the new module that we have just created in Admin Dashboard.</p></figcaption></figure>

Enabling a module in Orchard Core is a critical step, as it activates the features and services defined within the module. Orchard Core will detect the `IDataMigration` implementation and execute its `Create` or `Update` methods as needed. We will talk about this more in the Steps 3 and 4 below.

### Step 3: Define the Content Part

Firstly, we will need to create a part to hold the fields we defined for Product Information. Here's how we can create the `ProductInformationPart.cs` in `Models` folder.

```csharp
using OrchardCore.ContentManagement;

namespace OCBC.ProductModule.Models
{
    public class ProductInformationPart : ContentPart
    {
        public string ProductName { get; set; }
        public string ChineseProductName { get; set; }
        public string Model { get; set; }
        public string Description { get; set; }
        public string[] ProductImages { get; set; }
    }
}
```

Secondly, we have to register the part in the `Startup.cs` of our module:

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Routing;
using Microsoft.Extensions.DependencyInjection;
using OCBC.ProductModule.Models;
using OrchardCore.ContentManagement;
using OrchardCore.Modules;

namespace OCBC.ProductModule;

public sealed class Startup : StartupBase
{
    public override void ConfigureServices(IServiceCollection services)
    {
        services.AddContentPart<ProductInformationPart>();
    }

    ...
}
```

### Step 4: Create a Migration Class

In Orchard Core, a Migration class is a special kind of class that helps us define how our content types and fields should be structured in the system, and how they should look in the database.

Normally, when we want to add a new content type or field (like the Product Name or Product Description), we need to define them in the CMS. Instead of doing this manually every time, we write it in a Migration class, which allows Orchard Core to automatically create or update these content types and fields for us.

As your project grows, the structure of your content types might change. For example, you might want to add a new field or change how things work. Using a migration class helps track changes to the content types in a safe and organised way.

The migration class also takes care of the database. Orchard Core will automatically update the database to reflect the content types and fields you've defined. This means we do not need to manually create tables or fields in the database. Orchard Core handles all of that for us.

Assume for each of the Product Information records, we have the following fields.

* Product Name (one line);
* Chinese Product Name (one line);
* Model (selectable from a list of available models);
* Description (long text);
* Product Images (three images).

Before we proceed, we have to make sure the following package is installed to our module.

```bash
dotnet add package OrchardCore.ContentFields
```

In our module `OCBC.ProductModule,we`create a `Migrations` folder and add a class called `ProductInformationMigration.cs`.

```csharp
using OrchardCore.ContentManagement.Metadata;
using OrchardCore.Data.Migration;
using OrchardCore.ContentFields.Settings;
using OrchardCore.ContentManagement.Metadata.Settings;

namespace OCBC.ProductModule.Migrations
{
    public class ProductInformationMigration(IContentDefinitionManager contentDefinitionManager) : DataMigration
    {
        public int Create()
        {
            // Define the Product Information content type
            contentDefinitionManager.AlterTypeDefinitionAsync("ProductInformation", type => type
                .WithPart("CommonPart")  // Common Part that includes common fields (Author, Created, Modified)
                .WithPart("ProductInformationPart") // Custom Part (that we will define below)
            );

            // Define the ProductInformationPart with fields
            contentDefinitionManager.AlterPartDefinitionAsync("ProductInformationPart", part => part
                .WithField("ProductName", field => field
                    .OfType("TextField")
                    .WithDisplayName("Product Name"))
                .WithField("ChineseProductName", field => field
                    .OfType("TextField")
                    .WithDisplayName("产品名称"))
                .WithField("Model", field => field
                    .OfType("ContentPickerField")
                    .WithDisplayName("Model")
                    .WithSettings(new TextFieldSettings
                    {
                        Hint = "Please choose a model"
                    }))
                .WithField("Description", field => field
                    .OfType("TextField")
                    .WithDisplayName("Description")
                    .WithSettings(new TextFieldSettings
                    {
                        Hint = "Please provide a description"
                    }))
                .WithField("ProductImages", field => field
                    .OfType("MediaField")
                    .WithDisplayName("Product Images"))
            );

            return 1;
        }
    }
}
```

{% hint style="info" %}
To remove a field that was added in the previous migration, we need to do the following.

```csharp
contentDefinitionManager.AlterPartDefinitionAsync("ProductInformationPart", part => part
    .RemoveField("ProductName")
    .RemoveField("ChineseProductName")
    .RemoveField("Model")
    .RemoveField("Description")
);
```
{% endhint %}

The `return` value in a migration method represents the **version number** of the migration. Orchard Core uses this version to track the state of our module database schema and to determine which migration steps have been applied.

When we first define our `Create` method, we must return `1`. This indicates that our module is at **migration version 1** after the `Create` method is run.

For subsequent updates, when we need to make changes or additions to your content types, we define a new method named `UpdateFrom1`, `UpdateFrom2`, and so on. Each `UpdateFromX` method should return a version number that increments sequentially, as demonstrated in the code below.

```csharp
public int UpdateFrom1()
{
    // If you need to add more fields or changes later, this is where you'd do it.
    return 2;
}
```

If a module is currently at version 1 (based on what is stored in the database), and you deploy an updated module with an `UpdateFrom1` method, Orchard Core will automatically call that method to bring the database up to version 2.

Providing the **wrong number** in the migration methods can lead to unexpected behaviour in Orchard Core. Here’s what might happen and how to handle it. For example, if we intended to return 2 but we returned 10, then any future `UpdateFromX` migrations for versions 2 through 9 will **never run** because Orchard Core assumes they’ve already been applied.

Finally, please make sure services like `ProductInformationPart` and `ProductInformationMigration` are registered with the Dependency Injection (DI) container. Also, kindly make sure the `OCBC.ProductModule` has been enabled in the Admin Dashboard. This is to enable Orchard Core to detect the `IDataMigration` implementation (`ProductInformationMigration`) and execute its `Create` or `Update` methods as needed.

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption><p>The fields we defined in <code>ProductInformationPart.cs</code> will appear here.</p></figcaption></figure>

### Content Type Options

Content Type Options define how content types behave and interact with the system and UI.

* **Creatable** determines if an instance of this content type can be created through the UI.&#x20;
* **Listable** determines if an instance of this content type can be listed through the UI.&#x20;
* **Draftable** determines if this content type supports draft versions.&#x20;
* **Versionable** determines if this content type supports versioning.&#x20;
* **Securable** determines if this content type can have custom permissions.

For example, if we would like to make our ProductInformation content type to be cretable, listable, and draftable, then we can perform a migration update as follows.

```csharp
public int UpdateFrom2()
{
    contentDefinitionManager.AlterTypeDefinitionAsync("ProductInformation", type => type
        .Creatable()
        .Listable()
        .Draftable()
        .Versionable()
    );

    return 3;
}
```

With the migration above, after we perform `dotnet run`, we should be able to see three options have been checked, as shown below.

<figure><img src="../.gitbook/assets/image (45).png" alt=""><figcaption><p>Creatable, Listable, Draftable, and Versionable options are enabled now for Product Information.</p></figcaption></figure>

If we would like to uncheck the options, what we need to do is simply setting it false in the migration, as shown below.

```csharp
public int UpdateFrom3()
{
    contentDefinitionManager.AlterTypeDefinitionAsync("ProductInformation", type => type
        .Creatable(false)
    );

    return 4;
}
```

