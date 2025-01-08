# Workflow

In Orchard Core, workflows, webhooks, and other custom actions can be triggered by events such as publishing or unpublishing a content item. These capabilities are part of Orchard Core's extensibility and event-driven architecture.

Think of a workflow like a recipe for tasks we want to happen automatically in our CMS. For example:

* When a user fills out a form, send them an email.
* When we create a new content item, notify someone.
* Schedule a task to run at a specific time.

A workflow has two main parts:

1. **Trigger**: This is what starts the workflow, like "Content Item Created" or "Form Submitted".
2. **Tasks**: These are the steps that happen after the trigger, like "Send Email" or "Show Notification".

### Create Workflow via Admin Dashboard

For the simplicity of our demo, let's say we want to write a log every time a content item is created.

Before we begin, we need to make sure the `OrchardCore.Workflows` feature is enabled.

To create a new workflow, please visit the **Configuration > Workflows** in the admin menu.

After clicking on **New Workflow**, we can give our workflow a name, like "Log on Content Creation".

We then need to **Save** to proceed.

<figure><img src="../.gitbook/assets/image (49).png" alt=""><figcaption><p>Creating a new workflow.</p></figcaption></figure>

After creating the workflow, we need to add a new trigger. Here, we choose "Create Content" as the trigger. This means the workflow will start whenever a new content item is created.

<figure><img src="../.gitbook/assets/image (50).png" alt=""><figcaption><p>Added a trigger to our workflow.</p></figcaption></figure>

Next, we click Add Task to add a task. Choose "Log" as the task.

<figure><img src="../.gitbook/assets/image (51).png" alt=""><figcaption><p>Adding a new log task.</p></figcaption></figure>

Finally, we need to connect the Content Created Trigger to the Send Email activity. This tells Orchard Core, "When a content item of Product Information is created, log a Information message."

<figure><img src="../.gitbook/assets/image (52).png" alt=""><figcaption><p>Connecting the trigger to a task.</p></figcaption></figure>

We also need to indicate the trigger as our Startup Event by clicking on it and select the "power icon" to mark it as startup event, as pointed out in the screenshot below.

<figure><img src="../.gitbook/assets/image (53).png" alt=""><figcaption><p>Setting the trigger as startup event.</p></figcaption></figure>

As shown in the screenshot above, in order for a workflow to execute, at least one activity must be marked as the start of the workflow. Only triggers can be marked as the start of a workflow. A workflow can have more than one start event. This allows us to trigger a workflow in response to various types of events.

### Test of Workflow Trigger

Now, when we create a new Product Information content item, a log will be printed out in the console, as shown below.

<figure><img src="../.gitbook/assets/image (55).png" alt=""><figcaption><p>The informational log is printed by our workflow task successfully.</p></figcaption></figure>

In fact, we can also head to the Instances page to view the history of workflow executions.

<figure><img src="../.gitbook/assets/image (56).png" alt=""><figcaption><p>Each instance represents a single execution of the workflow.</p></figcaption></figure>

The Text field of the log event accepts Liquid. Let's use Liquid template to enhance it so that we know who created the content item. By default, [the Liquid templates have access to a common set of objects](https://docs.orchardcore.net/en/main/reference/modules/Liquid/#properties). In addition, there are [other properties available by default to any activity that supports Liquid expressions](https://docs.orchardcore.net/en/main/reference/modules/Workflows/#liquid-expressions). So, we can do something as follows.

```
This is a log from the Workflow for Product Information Creation: 
{{ Workflow.Input.ContentItem | json }} - User: {{ User.Identity.Name }}.
```

<figure><img src="../.gitbook/assets/image (58).png" alt=""><figcaption><p>The informational log is printed with the content item info and the user name.</p></figcaption></figure>

### Start the Workflow Programatically

In Orchard Core, we can use `StartWorkflowAsync` to run workflows. This method starts a new workflow instance based on a given workflow type.

Firstly, we retrieve a list of available workflow types which is available through `IWorkflowTypeStore`.

```csharp
var workflowTypes = await workflowTypeStore.ListAsync();
```

For our example above, since we only have one workflow, the method above should return the following JSON.

```json
"workflowTypes": [
    {
        "id": 52,
        "workflowTypeId": "4y8rkbh0xbbchrd9k26tdswpnd",
        "name": "Log on Content Creation",
        "isEnabled": true,
        "isSingleton": false,
        "lockTimeout": 0,
        "lockExpiration": 0,
        "deleteFinishedWorkflows": false,
        "activities": [
            {
                "activityId": "4yshv2dkk07s96v69yg1d94rwe",
                "name": "ContentCreatedEvent",
                "x": 0,
                "y": 60,
                "isStart": true,
                "properties": {
                    "ContentTypeFilter": [
                        "ProductInformation"
                    ],
                    "ActivityMetadata": {
                        "Title": "Product Information Creation"
                    }
                }
            },
            {
                "activityId": "4fpj20x90qtmkss8jgdzh0sgna",
                "name": "LogTask",
                "x": 450,
                "y": 50,
                "isStart": false,
                "properties": {
                    "ActivityMetadata": {
                        "Title": "Product Information Creation Log"
                    },
                    "LogLevel": "Information",
                    "Text": {
                        "Expression": "This is a log from the Workflow for Product Information Creation: {{ Workflow.Input.ContentItem | json }} - User: {{ User.Identity.Name }}."
                    }
                }
            }
        ],
        "transitions": [
            {
                "id": 0,
                "sourceActivityId": "4yshv2dkk07s96v69yg1d94rwe",
                "sourceOutcomeName": "Done",
                "destinationActivityId": "4fpj20x90qtmkss8jgdzh0sgna"
            }
        ],
        "properties": {}
    }
]
```

Hence, for demo purpose, let's see how we can start this workflow which has the id 52.

```csharp
// ...
using OrchardCore.Workflows.Services;

namespace OCBC.HeadlessCMS.Controllers;

[ApiController]
[Route("api/v1/product")]
public class ProductController(
    // ...
    IWorkflowTypeStore workflowTypeStore,
    IWorkflowManager workflowManager) : Controller
{
    // ...
    
    [HttpGet("trigger-workflow/{contentItemId}")]
    public async Task<IActionResult> TriggerWorkflowDemo(string contentItemId)
    {
        // Load the workflow definition.
        var workflowType = await workflowTypeStore.GetAsync(52);

        if (workflowType == null)
        {
            return BadRequest(new { Error = "Workflow not found." });
        }

        var productInformation = await orchard.GetContentItemByIdAsync(contentItemId);

        var input = new Dictionary<string, object>()
        {
            { "ContentItem", productInformation },
        };

        // Invoke the workflow.
        var workflowContext = await workflowManager.StartWorkflowAsync(workflowType, input: input);

        return Ok(new { workflowTypes, workflowContext });
    }
}
```

When we visit this endpoint, we shall see a new instance being created in the execution history of this workflow.

### HTTP Workflow

Orchard Core provides HTTP-related task which allows us to perform a HTTP request to a given URL through `OrchardCore.Workflows.Http`.

Before we begin, we need to make sure the `OrchardCore.Workflows.Http` feature is enabled.

Let's create a HTTP request task as shown below.

<figure><img src="../.gitbook/assets/image (59).png" alt=""><figcaption><p>This simple task is just to call Beeceptor mock server.</p></figcaption></figure>

The URL used in this example is Beeceptor, a mock API service. Beeceptor is a free, lightweight service designed for testing, mocking, and intercepting HTTP APIs. It is an excellent tool for quickly setting up endpoints to simulate or test API requests without needing to build a backend service.

After that, we link this HTTP request task with the Log task we created above, as demonstrated in the following screenshot.

<figure><img src="../.gitbook/assets/image (60).png" alt=""><figcaption><p>The workflow will now call the Beeceptor endpoint.</p></figcaption></figure>

So when this workflow is started, we shall see not just a log printed in the console, but the Beeceptor will detect that the endpoint `/orchard-core` is called.

<figure><img src="../.gitbook/assets/image (61).png" alt=""><figcaption><p>This show that an external endpoint on Beeceptor is successfully called from Orchard Core workflow.</p></figcaption></figure>
