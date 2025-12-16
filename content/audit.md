# Audit

Orchard Core has built-in features that can help with auditing and tracking changes, particularly for content and users.

To fully utilize auditing for both user-related actions and other general activities, we need to enable both `OrchardCore.AuditTrail` and `OrchardCore.Users.AuditTrail`.

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>These two Audit Trail features need to be enabled to ensure comprehensive auditing.</p></figcaption></figure>

### Configure Audit Trail

After the audit trail features are enabled, we need to configure it by specifying which events need to be recorded.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>Enabling Product Information content events to be recorded.</p></figcaption></figure>

Orchard Core supports audit trimming, which allows us to specify a retention period for audit events. This means we can configure how long audit data should be kept before it is automatically deleted or "trimmed." This helps keep the audit trail database from growing too large over time, especially in high-traffic applications.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption><p>The default retention period is 10 days.</p></figcaption></figure>

Audit Trail module supports tracking various events for Content Items. These events provide a comprehensive audit trail of changes and actions performed on content, which is useful for tracking modifications, maintaining an activity log, and ensuring compliance.

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption><p>At the end of the list, we can enable the recording of client IP address.</p></figcaption></figure>

### About Audit Trail

After we have configured the logging of Product Information, when we perform any of the enabled events, there should be a record logged.

We can view the logs under the Audit Trail section in the admin dashboard. The logs will be categorised, making it easy to distinguish between general activities and user-related actions.

<figure><img src="../.gitbook/assets/image (41).png" alt=""><figcaption><p>This shows the log after we publish one of the Product Information content items.</p></figcaption></figure>

Clicking on the `View` button or directly on the version, i.e. `Version 1`, it will display the content of the Product Information at that version.

<figure><img src="../.gitbook/assets/image (42).png" alt=""><figcaption><p>The content of Product Information at the moment of logging will be kept.</p></figcaption></figure>

Clicking on the `Details` button, we can view the detail of the Audit log.

<figure><img src="../.gitbook/assets/image (43).png" alt=""><figcaption><p>Details of the audit trail.</p></figcaption></figure>

Orchard Core also offers a page showing the line-by-line differences between two changes with its **"**&#x44;iff Vie&#x77;**"**. It compares changes between the two audit events.

<figure><img src="../.gitbook/assets/image (44).png" alt=""><figcaption><p>In the <code>Diff</code> tab, we will be able to see which part of the content has been edited.</p></figcaption></figure>

### Record Event Programatically

Let's assume now we are recording a new event for our Product Information, we will first need to retrieve the corresponding content item with its `contentItemId`.

```csharp
var productInformation = await orchard.GetContentItemByIdAsync(contentItemId);
```

Then, we can use the `IAuditTrailManager` service to record the event, as shown below.

```csharp
await auditTrailManager.RecordEventAsync(
    new AuditTrailContext<AuditTrailContentEvent>
    (
        name: "Published",
        category: "Content",
        correlationId: Guid.NewGuid().ToString(),
        userId: "4gyfgah3k3ness5adpmffgntnc",
        userName: "ADMIN",
        auditTrailEventItem: new AuditTrailContentEvent
        {
            ContentItem = productInformation,
            VersionNumber = 1,
            Comment = $"This is created at {DateTime.Now:yyyy-MM-dd HH:mm:ss}."
        }
    )
);
```

What is shown above is just a sample. The `name` can only be one of the enabled Events, such as Created, Saved, Published, etc. Otherwise the audit log will not be saved to the database.

The `category` must be "Content" since we are recording audit log for a content item. For user audit, it will be "User".

A `correlationId`refers to a unique identifier used to group or track related events across in our CMS. It serves as a common reference point to associate events that are part of the same workflow or transaction, making it easier to trace and diagnose issues.

The `userId` and `userName` in the sample above refer to the demo Admin account for demo purpose. You should change it to use the user ID and user name of the current user.

For the `VersionNumber`, if the `Versionable` is not enabled in the option of the content type, it should always be `1`. Otherwise, we can use the following to retrieve the current version of a content item.

```csharp
int version = (await contentManager.GetAllVersionsAsync(contentItemId)).Count();
```

The code basically will just count the number of entries in the database with the same `contentItemId`, as demonstrated below.

<figure><img src="../.gitbook/assets/image (47).png" alt=""><figcaption><p>The current version of content item with the ID "4fdpjjm5cgqz3y5zzaz72ccp4a" is 22 now.</p></figcaption></figure>
