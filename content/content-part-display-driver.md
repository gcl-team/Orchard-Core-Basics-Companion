# Content Part Display Driver

In Orchard Core, the `ContentPartDisplayDriver` is a powerful mechanism to control the rendering and editing of content parts. A driver offers advanced functionality such as dynamic behaviours, reusability, and backend-driven customisations.

### Setup View Model

We need to have a view model setup to isolate the data preparation logic from the `ContentPart` and display driver. The view only interacts with the view model making it easier to handle complex data structures. A view model is an excellent way to handle validation in Orchard Core. By applying **data annotations** (such as `[Required]`, `[StringLength]`, `[Range]`, etc.) to the properties of your ViewModel, you can validate user input in a clean and structured way.

For example, we can have the view model for our Product Information as follows under `/ViewModels/` in the module folder.

```csharp
using System.ComponentModel.DataAnnotations;
using OrchardCore.Media.Fields;

namespace OCBC.ProductModule.ViewModels;

public class ProductInformationPartViewModel
{
    [Required]
    public string ProductName { get; set; }
    [Required]
    public string ChineseProductName { get; set; }
    [Required]
    public string Model { get; set; }
    public string Description { get; set; }
    public MediaField[] ProductImages { get; set; }
}
```

In the code above, we are using MediaField, which is. Hence we will need to add the following NuGet package.

```bash
dotnet add package OrchardCore.Media
```

### Setup Display Driver

The `ContentPartDisplayDriver` in Orchard Core primarily focuses on handling the **display**, **edit**, and **update** workflows for a content part.

* **Display** handles the rendering of the part when the content item is viewed;
* **Edit** handles the rendering of the part when the content item is being edited;
* **UpdateAsync** handles the processing and updating of the part when the form is submitted.

To enable our Content Part using the display driver, we need to tell it which display driver to use in the configuration of the module with `UseDisplayDriver`.

```csharp
using OrchardCore.ContentManagement.Display.ContentDisplay;
//...

namespace OCBC.ProductModule;

public sealed class Startup : StartupBase
{
    public override void ConfigureServices(IServiceCollection services)
    {
        services.AddContentPart<ProductInformationPart>()
            .UseDisplayDriver<ProductInformationPartDisplayDriver>();
        //...
    }

    //...
}
```

After that, we can proceed to create a new display driver class for our Product Information Part called `ProductInformationPartDisplayDriver`. It can be placed in the `Drivers` folder in the module.

### Edit View

For the display driver, we will setup the Edit part as follow.

```csharp
using OCBC.ProductModule.Models;
using OCBC.ProductModule.ViewModels;
using OrchardCore.ContentManagement.Display.ContentDisplay;
using OrchardCore.ContentManagement.Display.Models;
using OrchardCore.DisplayManagement.Views;

public class ProductInformationPartDisplayDriver : ContentPartDisplayDriver<ProductInformationPart>
{
    public override IDisplayResult Edit(ProductInformationPart part, BuildPartEditorContext context) =>
        Initialize<ProductInformationPartViewModel>(
            GetEditorShapeType(context),
            viewModel => PopulateViewModel(part, viewModel))
        .Location("Content:5");

    private static void PopulateViewModel(ProductInformationPart part, ProductInformationPartViewModel viewModel)
    {
        viewModel.ProductName = part.ProductName;
        viewModel.ChineseProductName = part.ChineseProductName;
        viewModel.Model = part.Model;
        viewModel.Description = part.Description;
    }
}

```

This method defines how the `Edit` interface for a `ContentPart` (in this case, `ProductInformationPart`) is rendered in the Administration UI. It is triggered when the system needs to build the editing interface for the `ProductInformationPart`. Here, the view model  `ProductInformationPartViewModel` bridges the content type `ProductInformationPart` and the Razor view.

The method `GetEditorShapeType(context)` dynamically determines the name of the shape to use for rendering. Typically it resolves to `<PartName>.Edit.cshtml` under `/Views/` in the module folder.

In Orchard Core, a **"**&#x73;hap&#x65;**"** is a key concept in the display management system. It represents a flexible and dynamic UI component that can be rendered as part of our content.

Finally, the `Location("Content:5")` is to specify where the shape should be rendered in the UI. In this case, we use `Content`, which indicates the zone in which the shape will be rendered while the `5` after it specifies the priority or order in which the shape appears relative to other shapes in the same zone. **Lower numbers render first.**

With this setup, we can setup the Razor view for the Edit view. What we need to do is create a `ProductInformationPart.Edit.cshtml` file under `Views` folder with the following code.

```cshtml
@model OCBC.ProductModule.ViewModels.ProductInformationPartViewModel

<div class="mb-3 row">
    <!-- Product Name -->
    <div class="col-md-6">
        <div class="form-group">
            <label asp-for="ProductName" class="control-label">Product Name</label>
            <input asp-for="ProductName" class="form-control" />
            <span asp-validation-for="ProductName" class="text-danger"></span>
        </div>
    </div>

    <!-- Chinese Product Name -->
    <div class="col-md-6">
        <div class="form-group">
            <label asp-for="ChineseProductName" class="control-label">产品名称</label>
            <input asp-for="ChineseProductName" class="form-control" />
            <span asp-validation-for="ChineseProductName" class="text-danger"></span>
        </div>
    </div>
</div>

<div class="mb-3 row">
    <!-- Model -->
    <div class="col-md-12">
        <div class="form-group">
            <label asp-for="Model" class="control-label">Model</label>
            <input asp-for="Model" class="form-control" />
            <span asp-validation-for="Model" class="text-danger"></span>
        </div>
    </div>
</div>

<div class="mb-3 row">
    <!-- Description -->
    <div class="col-md-12">
        <div class="form-group">
            <label asp-for="Description" class="control-label">Description</label>
            <textarea asp-for="Description" class="form-control" rows="4"></textarea>
            <span asp-validation-for="Description" class="text-danger"></span>
        </div>
    </div>
</div>
```

The code above should render the following UI.

<figure><img src="../.gitbook/assets/image (27).png" alt=""><figcaption><p>The Edit view for the Product Information.</p></figcaption></figure>

As shown in the screenshot above, the Product Images field is not in our `ProductInformationPart.Edit.cshtml` but how come the image field is still rendered? In Orchard Core, even if we do not explicitly add HTML for a field in our `Edit.cshtml`, the field can still render automatically if it is registered via the `WithField` method in the `DisplayDriver`. This behaviour occurs because Orchard Core uses the **Shape system** to render fields.

When we submit a form with data, the submitted values are sent to the server as part of the HTTP POST request. However, this raw data will not automatically populate our server-side `ViewModel`. Hence, we need to extract the values from the request and map them to the correct model properties. This is where `context.Updater.TryUpdateModelAsync` comes into play in the display driver.

```csharp
public class ProductInformationPartDisplayDriver : ContentPartDisplayDriver<ProductInformationPart>
{
    public override async Task<IDisplayResult> UpdateAsync(ProductInformationPart part, UpdatePartEditorContext context)
    {
        var viewModel = new ProductInformationPartViewModel();
        
        // Bind using the correct prefix
        if (await context.Updater.TryUpdateModelAsync(viewModel, Prefix))
        {
            part.ProductName = viewModel.ProductName;
            part.ChineseProductName = viewModel.ChineseProductName;
            part.Model = viewModel.Model;
            part.Description = viewModel.Description;
            part.ProductImage1 = viewModel.ProductImage1;
            part.ProductImage2 = viewModel.ProductImage2;
            part.ProductImage3 = viewModel.ProductImage3;
            
            var contentItem = part.ContentItem;
            contentItem.DisplayText = $"{viewModel.Model} - {viewModel.ProductName}";
        }
        
        return await EditAsync(part, context);
    }
}

```

The concept of "binding using the correct prefix" comes into play here where we have multiple models or parts on a page, and we want to ensure that form values are bound correctly to the right model properties.

Before leaving the method, we also update the DisplayText so that the listing of Content Items on Administration UI will give us useful names instead of the default "Product Information", which is the Content Type name.

<figure><img src="../.gitbook/assets/image (24).png" alt=""><figcaption><p>Listing of content items with user-friendly descriptive names.</p></figcaption></figure>

Finally, we must save the changes to the database asynchronously, which is done with the `EditAsync`.

### Edit View - More MediaFields

It is not fun with just one product image. Let's allow users to upload three images for each of the product.

Firstly, we update the migration as follows (Your migration number may be different).

```csharp
public int UpdateFrom10()
{
    contentDefinitionManager.AlterPartDefinitionAsync("ProductInformationPart", part => part
        .RemoveField("ProductImages")
        .WithField("ProductImage1", field => field
            .OfType("MediaField")
            .WithDisplayName("Product Image 1")
            .WithPosition("Editor:1"))
        .WithField("ProductImage2", field => field
            .OfType("MediaField")
            .WithDisplayName("Product Image 2")
            .WithPosition("Editor:2"))
        .WithField("ProductImage3", field => field
            .OfType("MediaField")
            .WithDisplayName("Product Image 3")
            .WithPosition("Editor:3"))
        );
            
    return 11;
}
```

Secondly, we will update our display driver as follows.

```csharp
public class ProductInformationPartDisplayDriver : ContentPartDisplayDriver<ProductInformationPart>
{
    // ...

    public override async Task<IDisplayResult> UpdateAsync(ProductInformationPart part, UpdatePartEditorContext context)
    {
        var viewModel = new ProductInformationPartViewModel();
        
        // Bind using the correct prefix
        if (await context.Updater.TryUpdateModelAsync(viewModel, Prefix))
        {
            // ...
            part.ProductImage1 = viewModel.ProductImage1;
            part.ProductImage2 = viewModel.ProductImage2;
            part.ProductImage3 = viewModel.ProductImage3;

            // ...
        }
        
        return await EditAsync(part, context);
    }

    private static void PopulateViewModel(ProductInformationPart part, ProductInformationPartViewModel viewModel)
    {
        // ...
        viewModel.ProductImage1 = part.ProductImage1;
        viewModel.ProductImage2 = part.ProductImage2;
        viewModel.ProductImage3 = part.ProductImage3;
    }
}
```

Thirdly, before we proceed to update the Edit view, we will find out that the three MediaFields have already been displayed.

<figure><img src="../.gitbook/assets/image (22).png" alt=""><figcaption><p>Default MediaFields are displayed.</p></figcaption></figure>

We can customise them with CSS in the cshtml file so that the three MediaFields appear side by side.

```html
<style>
    .content-part-wrapper-product-information-part {
        display: flex;
        gap: 10px; /* Adds space between the divs */
    }

    .content-part-wrapper-product-information-part .field-wrapper {
        flex: 1;                 /* Each field takes up equal space */
        max-width: 33%;          /* Ensure no field exceeds 1/3 of the container */
        display: block;          /* Ensure proper block-level display */
        box-sizing: border-box;  /* Ensure padding and borders don't affect width */
        padding: 10px;
    }

    .content-part-wrapper-product-information-part {
        display: flex;            /* Use flexbox for side-by-side layout */
        gap: 10px;                /* Add spacing between the fields */
        flex-wrap: wrap;          /* Allow wrapping if necessary */
        border: 1px solid #ccc;   /* Add border for the fieldset */
        padding: 10px;
        position: relative;       /* Required for positioning the pseudo-elements */
    }

    /* Create a 'fieldset' around the fields using pseudo-elements */
    .content-part-wrapper-product-information-part::before {
        content: "";              /* Create an empty content for the pseudo-element */
        position: absolute;
        top: -10px;
        left: 0;
        right: 0;
        bottom: 100%;
        border-radius: 5px;
        background: transparent;
        padding: 10px;
        z-index: -1;              /* Ensure the pseudo-element is behind the content */
    }

    /* Create a 'legend' label for the fieldset */
    .content-part-wrapper-product-information-part::after {
        content: "Product Gallery"; /* Text for the 'legend' */
        position: absolute;
        top: -12px;
        left: 10px;
        font-weight: bold;
        border-radius: 6px;
        background: #0d6efd;
        padding: 0 5px;
    }
</style>
```

We will then get the following UI for the Edit.

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption><p>Updated UI for the Edit page of Product Information.</p></figcaption></figure>

### Display View

We can continue to setup our display driver for the Display view, as shown below.

```csharp
public class ProductInformationPartDisplayDriver : ContentPartDisplayDriver<ProductInformationPart>
{
    public override IDisplayResult Display(ProductInformationPart part, BuildPartDisplayContext context)=>
        Initialize<ProductInformationPartViewModel>(
            GetDisplayShapeType(context),
            viewModel => PopulateViewModel(part, viewModel))
        .Location("Detail", "Content:5");
    
    //...
}

```

The `Display` method creates the display logic for rendering our `ProductInformationPart` in the Details context.

We thus need to create a `ProductInformationPart.Detail.cshtml` file under `Views` folder with the following code.

```cshtml
@model OCBC.ProductModule.ViewModels.ProductInformationPartViewModel

<div class="product-information">
    <!-- Product Header Section -->
    <div class="product-header text-center mb-4">
        <h1 class="product-title">Product Info: @Model.ProductName</h1>
        <h2 class="product-subtitle text-muted">@Model.ChineseProductName</h2>
    </div>

    <!-- Product Details Section -->
    <div class="product-details">
        <div class="row mb-3">
            <div class="col-md-4 text-end fw-bold">Model:</div>
            <div class="col-md-8">@Model.Model</div>
        </div>
        <div class="row">
            <div class="col-md-4 text-end fw-bold">Description:</div>
            <div class="col-md-8">@Html.Raw(Model.Description ?? "<em>No description provided.</em>")</div>
        </div>
    </div>

    <!-- Product Images Section -->
    @{
        var imagesForColumns = new[]
        {
            new List<string>(),
            new List<string>(),
            new List<string>(),
            new List<string>()
        };
        var allImagePaths = (Model.ProductImage1?.Paths ?? [])
            .Concat((Model.ProductImage2?.Paths ?? []))
            .Concat((Model.ProductImage3?.Paths ?? []))
            .ToArray();
        for (var i = 0; i < allImagePaths.Length; i++)
        {
            var imageUrl = allImagePaths[i];

            imagesForColumns[i % 4].Add(imageUrl);
        }
    }
    @if (allImagePaths.Count() > 0)
    {
        <div class="image-gallery-row">
            <div class="image-gallery-column">
                @foreach(var imageUrl in imagesForColumns[0])
                {
                    <img src="/media/@imageUrl" style="width:100%" />
                }
            </div>
            <div class="image-gallery-column">
                @foreach(var imageUrl in imagesForColumns[1])
                {
                    <img src="/media/@imageUrl" style="width:100%" />
                }
            </div>
            <div class="image-gallery-column">
                @foreach(var imageUrl in imagesForColumns[2])
                {
                    <img src="/media/@imageUrl" style="width:100%" />
                }
            </div>
            <div class="image-gallery-column">
                @foreach(var imageUrl in imagesForColumns[3])
                {
                    <img src="/media/@imageUrl" style="width:100%" />
                }
            </div>
        </div>
    }
    else
    {
        <p class="text-muted text-center">No product images available.</p>
    }
</div>

<style>
    .field {
        display: none;
    }
    .product-information {
        max-width: 800px;
        margin: 0 auto;
        font-family: 'Arial', sans-serif;
    }
    .product-title {
        font-size: 2rem;
        font-weight: 700;
    }
    .product-subtitle {
        font-size: 1.25rem;
        font-weight: 400;
    }
    .product-details .row {
        padding: 0.5rem 0;
        border-bottom: 1px solid #eaeaea;
    }
    .product-details .row:last-child {
        border-bottom: none;
    }
    .image-gallery-row {
        display: flex;
        flex-wrap: wrap;
        padding: 0 4px;
    }

    /* Create four equal columns that sits next to each other */
    .image-gallery-column {
        flex: 23%;
        max-width: 23%;
        padding: 0 4px;
    }

    .image-gallery-column img {
        margin-top: 8px;
        vertical-align: middle;
        width: 100%;
    }

    /* Responsive layout - makes a two column-layout instead of four columns */
    @@media screen and (max-width: 800px) {
        .image-gallery-column {
            flex: 50%;
            max-width: 50%;
        }
    }

    /* Responsive layout - makes the two columns stack on top of each other instead of next to each other */
    @@media screen and (max-width: 600px) {
        .image-gallery-column {
            flex: 100%;
            max-width: 100%;
        }
    }
</style>
```

The code above will render the following web page when we are visiting one of the products.

<figure><img src="../.gitbook/assets/image (19).png" alt=""><figcaption><p>The Display view of a Product Info.</p></figcaption></figure>

