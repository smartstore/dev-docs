# 🥚 DbContextScope

* A `SmartDbContext` instance is short-lived
* An instance is obtained from the pool when a HTTP request begins, and returned to the pool when request ends
* To change the configuration of the request scoped `SmartDbContext` you can create a custom nested scope, do something within the scope...
* ...and when your scope disposes the context's original state (it had when you created the scope) is restored automatically
* A custom scope is useful in scenarios where the default configuration of the data context does not fit your needs
  * e.g. in long-running operations you may want to disable automatic change detection (which is slow when the change tracker contains many entities)
  * or you want to disable hook invocation to speed up write access (because hook resolution and invocation takes some time)
  * or you want to force your scope to be atomic (save all in batch when scope ends or you commit explicitly)
* _BASIC EXAMPLE_

## Parameter reference

* All parameters except `db` are optional
* When a param is passed with a non-null value, then the corresponding option will be changed for the duration of your scope.

| Parameter             | Description                                                                                             |
| --------------------- | ------------------------------------------------------------------------------------------------------- |
| `db`                  | The data context instance to create scope for                                                           |
| `autoDetectChanges`   | Toggles EF's automatic entity change detection feature                                                  |
| `lazyLoading`         | Toggles EF's lazy loading feature                                                                       |
| `forceNoTracking`     | If `true`, queries will be untracked by default, unless specified otherwise                             |
| `deferCommit`         | Suppresses the execution of `SaveChanges` until the scope is disposed or `Commit` is called explicitly. |
| `retainConnection`    | Opens connection and retains it until scope ends. May increase load/save performance in large scopes.   |
| `minHookImportance`   | Specifies the min hook importance level. Hooks below this level will not be executed.                   |
| `cascadeDeleteTiming` | Sets the change tracker's `CascadeDeleteTiming` option                                                  |
| `deleteOrphansTiming` | Sets the change tracker's `DeleteOrphansTiming` option                                                  |
