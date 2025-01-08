# Remote Deployment

Orchard Core supports moving content between sites using **Remote Deployment**, which is a feature designed to synchronize or deploy content, recipes, and other configurations between different Orchard Core environments or instances. The feature leverages APIs to send and receive deployment plans and packages.

### Key Components of Remote Deployment&#x20;

**Deployment Plans**: Deployment Plan is a feature in Orchard Core that lets us define and automate the process of exporting specific content from one Orchard Core instance (source) and importing them into another instance (target). It's essentially a blueprint for transferring data between environments, such as from a staging server to a production server. It is a collection of steps (tasks) that define what content or settings should be exported or deployed.&#x20;

**Deployment Targets**: A configuration that defines the destination site where the deployment will be sent.&#x20;

**Deployment Steps**: Specific tasks within a deployment plan, such as exporting content items, settings, or media.

If the built-in deployment steps don’t cover your needs, you can create custom deployment steps by extending Orchard Core.

### Setup

Since we have already enabled "OrchardCore.Contents.Deployment.ExportContentToDeploymentTarget", "OrchardCore.Deployment" and "OrchardCore.Deployment.Remote" features in the Recipe earlier, the remote deployment should have already been enabled.

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption><p>Please make sure the three features are enabled.</p></figcaption></figure>

Take note that both source and target servers must have the **Remote Deployment** module enabled.

In the following example, we will be using the following two local instances as source and target:

* Source server: http://localhost:5022/
* Target server: http://localhost:5000/

Now, we need to link them up by setting the source server as the Remote Client at the target server. To do so, we visit the Configuration > Import/Export > Remote Clients page. Then we click on the "Add Remote Client" button which will bring us to the following page.

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption><p>Adding our source server as the remote client.</p></figcaption></figure>

In the API Key field, we have to manually type in our desired API Key value (e.g., a long, secure, randomly generated string). Tools like password managers or online key generators can help generate strong keys.

Next, we need to copy the URL shown on the "Remote Clients" page because this is the URL we will be used later to tell our source server where our target server is located at.

<figure><img src="../.gitbook/assets/image (14).png" alt=""><figcaption><p>We need to note down the URL of the remote instance which will be used by the client instances.</p></figcaption></figure>

After setting up the target server, we need to head to the source server to set our target server as the Remote Instance. To do so, we visit the Configuration > Import/Export > Remote Instances page. Then we click on the "Add Remote Instance" button which will bring us to the following page.

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>Linking source and target servers with the URL and API key earlier.</p></figcaption></figure>

### Remote Deployment

Now let's pick an existing content item to deploy to the target server with the "Export to Deployment Target" option, as shown in the screenshot below.

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption><p>We can only choose to deploy a particular content item.</p></figcaption></figure>

Next, we will be asked where to deploy to.

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption><p>Choose the target server as the target of remote deployment.</p></figcaption></figure>

If the deployment is successful, there should be a success message shown. If the deployment fails, please make sure the earlier setup is done correctly.

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption><p>Deployment is success!</p></figcaption></figure>

Now, if we check the database of the target server, we will find our content item has been inserted successfully.

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption><p>The content is successfully exported to the target server.</p></figcaption></figure>
