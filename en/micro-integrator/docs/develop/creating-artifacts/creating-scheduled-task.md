# Scheduling ESB Tasks

Follow the instructions given below to create a Scheduled Task in WSO2 Integration Studio.

## Instructions

1.  If you have already created an [ESB Config project](../../creating-projects/#esb-config-project), right-click the project and click **New** → **Scheduled Task**.  
2.  Select **Create a New Scheduled Task Artifact** and click **Next**.
3.  Specify values for the [required parameter](../../references/synapse-properties/scheduled-task-properties.md) for the scheduled task.
4.   Do one of the following:  
    -   To save the scheduled task in an existing ESB Config project in your workspace, click **Browse** and select that project.
    -   To save the scheduled task in a new ESB Config project, click **Create a new ESB project** and create the new project.
5.  Click **Finish**. The scheduled task is created in the `src/main/synapse-config/tasks` folder under the ESB Config project you specified.
6.  Open the new artifact from the project explorer. In the **Form** view, click **Task Implementation Properties** and update any [optional scheduled task properties](../../references/synapse-properties/scheduled-task-properties.md).

## Examples

-   [Task Scheduling using a Simple Trigger](../../../use-cases/examples/scheduled-tasks/task-scheduling-simple-trigger)
-   [Injecting Messages to a RESTful Endpoint](../../../use-cases/examples/scheduled-tasks/injecting-messages-to-rest-endpoint)

## Tutorials

-   See the tutorial on [periodically executing an integration process](../../../use-cases/tutorials/using-scheduled-tasks) using a scheduled task
