# Schemas

In Orchard Core, the database schema is designed to support its modular and extensible architecture, with each module potentially introducing its own tables to manage specific functionalities.

<table><thead><tr><th width="262">Table</th><th>Description</th></tr></thead><tbody><tr><td>AliasPartIndex</td><td>This table indexes alias parts associated with content items. An alias part allows a content item to have a custom URL or identifier, facilitating user-friendly URLs and easy content referencing.</td></tr><tr><td>Audit_AuditTrailEventIndex</td><td>Part of the Audit Trail module, this table records events related to content items, such as creation, modification, and deletion. It helps in tracking changes and maintaining a history of actions performed within the CMS.</td></tr><tr><td>Audit_Document</td><td>Associated with the Audit Trail module, this table stores serialised audit trail events. It provides a detailed log of activities for auditing and monitoring purposes.</td></tr><tr><td>ContainedPartIndex</td><td>This table indexes content items that have a <strong>ContainedPart</strong> attached.</td></tr><tr><td>ContentItemIndex</td><td>Stores essential information about content items, such as their ID, type, version, status (latest, published), and creation/last modified timestamps.</td></tr><tr><td>DeploymentPlanIndex</td><td>A deployment plan in Orchard Core defines what content or settings should be exported or deployed.</td></tr><tr><td>Document</td><td>Stores serialized JSON documents representing various data in Orchard Core.</td></tr><tr><td>IndexingTask</td><td>The IndexingTask table is used to track tasks related to indexing content, typically for search functionality.</td></tr><tr><td>LayerMetadataIndex</td><td>Indexes metadata related to layers in Orchard Core. Layers are used for conditional rendering of widgets on specific parts of a site.</td></tr><tr><td>OpenID_*</td><td>Tables related to OpenID Connect support in Orchard Core. They are used to manage applications, tokens, roles, and URIs associated with OpenID Connect.<br>These tables collectively ensure that Orchard Core can efficiently manage OpenID Connect applications, URIs, and roles, supporting secure and standards-compliant authentication and authorisation workflows.</td></tr><tr><td>UserByClaimIndex</td><td>Indexes users by their claims.<br>A claim is a key-value pair associated with a user, often used in authentication or authorisation scenarios (e.g., a claim might specify the user's email or permissions).</td></tr><tr><td>UserByLoginInfoIndex</td><td>Indexes users based on their login information.</td></tr><tr><td>UserByRoleNameIndex</td><td>Indexes users based on the roles they are assigned to. Roles are a key part of Orchard Core role-based access control (RBAC) system.</td></tr><tr><td>UserByRoleNameIndex_Document</td><td>Provides a relationship between indexed role-based user entries and the full user metadata stored in the Document table.</td></tr><tr><td>UserIndex</td><td>The main table indexing users and their essential metadata. Contains fields such as user ID, username, email, and whether the user is enabled or disabled.</td></tr></tbody></table>

### Create Custom Tables

We can add our own table in Orchard Core. Adding custom tables usually involves creating a custom module and defining the table schema using Migrations.

To add a custom table, we typically define it using `SchemaBuilder` in a **Migrations** class in a custom Orchard Core module.

```csharp
public int UpdateFrom13()
{
    SchemaBuilder.CreateTableAsync("MyCustomTable", table => table
        .Column<int>("Id", col => col.PrimaryKey().Identity())
        .Column<string>("CustomField", col => col.WithLength(255))
        .Column<DateTime>("CreatedDate")
    );
            
    return 14;
}
```

The custom table defined above will be created as below.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption><p>Successfully created our own table MyCustomTable.</p></figcaption></figure>

### YesSql

{% hint style="info" %}
Want to learn about YesSql? Head to the [official YesSql tutorial](https://github.com/sebastienros/yessql/wiki/Tutorial#querying-documents) to learn more.
{% endhint %}

In Orchard Core, data is stored in a database, but instead of using a traditional approach like defining fixed tables and columns for every type of data, Orchard Core uses **YesSql**.

YesSql is a lightweight library that acts as a bridge between Orchard Core and the database. It lets Orchard Core store and retrieve data more flexibly, without needing to define a strict database structure in advance.

YesSql is a tool that helps Orchard Core handle data as **documents**. A document is like a package that can contain any kind of information, such as content items, settings, or user data. These documents are stored in the database in a generic way, which makes it easy for Orchard Core to adapt to different types of content without creating new database tables every time.

YesSql also provides a way to create **indexes**, which are like shortcuts to quickly find and organize data. For example, if Orchard Core needs to find all blog posts written by a specific author, an index can make that search faster. These indexes are defined in the code, so they can be customized for different needs.

Orchard Core is built on top of a relational database (like SQL Server, SQLite, or MySQL). YesSql maps documents to tables in the database, but the data in those tables is stored in a flexible, document-like format. The tables we discussed above are mostly used to store documents or indexes, allowing Orchard Core to function in a flexible, schema-less way while still using relational databases for storage.

In short, YesSql is the engine behind Orchard Core's database operations. It keeps things flexible, efficient, and easy to extend, allowing Orchard Core to support a wide variety of features and customisations. This is one of the reasons why Orchard Core is so powerful and adaptable.

### Model for Custom Table

Accessing the custom table `MyCustomTable` from our code in Orchard Core typically involves using the YesSql ORM that Orchard Core is built upon.

Firstly, we need to define a model class that maps to the schema of our custom table. The property names should match the column names in your table.

```csharp
namespace OCBC.ProductModule.Models;

public class MyCustomTable
{
    public int Id { get; set; }
    public string CustomField { get; set; }
    public DateTime CreatedDate { get; set; }
}
```

To access `MyCustomTable` using YesSql, we will use the `YesSql.ISession` service to interact with the database.

### Create Data in Custom Table

To insert a new row to `MyCustomTable`, we will `ISession.Save`.

```csharp
var newEntry = new MyCustomTable
{
    CustomField = customField,
    CreatedDate = DateTime.Now,
};

await session.SaveAsync(newEntry);
await session.SaveChangesAsync();
```

### Query Custom Table

We will use `ISession.Query` to retrieve data.

```csharp
await session.Query<MyCustomTable>().ListAsync();
```
