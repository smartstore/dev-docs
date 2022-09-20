---
description: Schedules automated tasks to be executed periodically
---

# Scheduling

* Executes automated tasks, e.g.: cleanup files, cleanup database, send emails, rebuild XML sitemap, periodic imports or exports etc.
* Perfect for long-running processes or expensive stuff.



* Is a **web** scheduler
* No timer that executes tasks, but...
* ...a timer polls the web scheduler HTTP endpoint every minute
* The endpoint determines all overdue tasks and executes them within the scope of a HTTP request
* This guarantees that the dependency scope is always up and running. No need to create custom scopes.
* The task context virtualizer (`ITaskContextVirtualizer`) will virtualize some environment parameters during task execution (unless specified otherwise):
  * `IWorkContext.CurrentCustomer` --> BackgroundTask system customer
  * `IStoreContext.CurrentStore` --> Primary (first) store

## Task descriptor

* The `TaskDescriptor` domain entity defines the task metadata: name, cron expression, whether it is enabled or not, priority, task type to execute etc.
* This entity is saved in database
* Some parts can be edited by the user in the backend, e.g. cron expression, enabled etc.
* A task's cron expression specifies the next run time
* When a task has run, a history entry is created containing infos about: time of execution, duration, name of machine that leased the execution, error etc.
* A task can also be triggered manually by the user in the backend
* Every running task can be cancelled explicitly in the backend

## Implementing a task

* Any concrete class implementing the `ITask` interface
* No need to register in DI, is auto-discovered on app startup
* `Run` method is the task handler. No sync-counterpart!
* The task executor will call this method asynchronously and await it
* Any exception raised during execution (either unhandled or explicity) will stop execution. The error will be logged. If the task descriptor's `StopOnError` property is `true`, task will be disabled and will not run anymore (unless user turns it on again).
* NOTE: if the task resolver determined more than one overdue task within a single poll operation, they will NOT be executed in parallel, but one after another. If a task's execution is not finished on the next poll (next minute), the executor will skip it.
* NOTE: Because of the minutely polling, it makes no sense to define cron expression with less fraction, e.g. "every 30 seconds".
* _SAMPLE_

## Task cancellation

* Every task should be cancellable, especially those that are long-running
* Remember that the user can request cancellation in the backend
* Therefore the `CancellationToken` parameter is passed. It combines app shutdown token and user cancellation token.
* Regularly check whether cancellation is requested and try to gracefully quit your task
* _Regular_ means: not necessarily in every iteration, bur after a batch of something completed.
* _SAMPLE_
* Or just throw if cancellation was requested
* _SAMPLE_ (ThrowIfCancellationRequested())

## Propagating task progress

* The task scheduler UI in backend can display task progress (either message, percent or both)
* But you must provide this info
* The `TaskExecutionContext` parameter passed to the `Run` method contains the `SetProgress` method (with different overloads and sync/async variants).
* Call it to propagate progress. Your progress is saved in database immediately.
* The UI fetches refreshed progress info every seconds and can now display it

## Adding or removing tasks programmatically

* Necessary if your module provides tasks
* In this case task must be added during module installation and removed during uninstallation

{% code title="Add and remove tasks programmatically" %}
```csharp
internal class Module : ModuleBase
{
    private readonly ITaskStore _taskStore;

    public Module(ITaskStore taskStore)
    {
        _taskStore = taskStore;
    }

    public override async Task InstallAsync(ModuleInstallationContext context)
    {
        // Add the task, if it does not exist yet.
        await _taskStore.GetOrAddTaskAsync<MyTaskImplType>(x =>
        {
            x.Name = "My localized task display name";
            x.CronExpression = "0 */1 * * *"; // every hour
            x.Enabled = true;
        });

        await base.InstallAsync(context);
    }

    public override async Task UninstallAsync()
    {
        // Try to remove the task. Do nothing if it does not exist.
        await _taskStore.TryDeleteTaskAsync<MyTaskImplType>();

        await base.UninstallAsync();
    }
}
```
{% endcode %}

## System tasks

* If you want to hide a task from the user, `TaskDescriptor.IsHidden` must be set to `true`.
* This prop cannot be edited by the user.
* A hidden task is not displayed in task scheduler UI.
* But you can invoke the `MinimalTaskViewComponent` to render a single task's state anywhere you want
* The component displays task common info in a very compact widget
* _SCREENSHOT_
* It displays progress info if the task is currently running + a cancel button, or
* it provides buttons to execute the task immediately and to edit the cron expression.

{% code title="Invoking the MinimalTask view component" %}
```cshtml
@await Component.InvokeAsync("MinimalTask", new 
{ 
	// The TaskDescriptor entity id
	taskId = Model.TaskId, 
	// "false" hides the "Cancel" button for a running task
	cancellable = false
})
```
{% endcode %}

## Executing single tasks programmatically

* `ITaskScheduler.RunSingleTaskAsync`

## Task leasing

* RunPerMachine

## Implementing a custom Task Store provider

* `ITaskStore`
* The default implementation stores task info and progress in database
* We cannot think of any scenario where it makes sense to override this :-)
* But for the sake of completeness:
* Create a class that implements `ITaskStore`
* Follow the contract and implement all members
* Register your custom class in service container --> will overwrite the default registration
* _SAMPLE_
