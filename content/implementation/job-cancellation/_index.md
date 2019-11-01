+++
title = "Job cancellation"
weight = 10
+++

Internally, the `execute` method calls the `launch` coroutine builder to fire off a coroutine Job. The scope for these coroutines will be a `CoroutineScope` contained by the ViewModel.

```kotlin
abstract class JobViewModel<VS : Any>(initialState: VS) 
    : RainbowCakeViewModel<VS>(initialState) {

    private val coroutineScope = CoroutineScope(Dispatchers.Main + SupervisorJob())
    
    // ...
}
```

The context of the scope is made up of two components: the UI dispatcher and a [`SupervisorJob`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-supervisor-job.html) instance that acts as the parent of all coroutines launched in the ViewModel.

- `Dispatchers.Main` describes the `CoroutineDispatcher` element to use, in this case, the Android UI thread until the job is placed on another dispatcher (for example, with a `withContext` call).
- `SupervisorJob()` provides the parent `Job` element for the coroutine. This is a simple, empty `Job`, only used to group the jobs launched inside a given ViewModel. This job is cancelled is cancelled (indirectly, by cancelling the scope itself) when the ViewModel's `onCleared` method is called. This happens when the lifecycle it was attached to has terminated, meaning the `Activity`/`Fragment` is actually destroyed for good, and not just going through configuration changes.

```kotlin
override fun onCleared() {
    super.onCleared()
    coroutineScope.cancel()
    log("ViewModel cleared, job cancelled")
}
```

Cancelling the root job will also cancel all of its child coroutines, which in this case is every coroutine launched with the `execute` method. Note that coroutine cancellation is cooperative. Blocking code running inside the coroutine will not be cancelled. Cancellation can only happen in suspending functions, and only if it's handled explicitly. [Retrofit's coroutine integration](/best-practices/retrofit-and-coroutines/) is a good example of this.

### References

The reasoning for using a `SupervisorJob` (as well as other good coroutine tips) is laid out nicely [in this article](https://proandroiddev.com/kotlin-coroutines-patterns-anti-patterns-f9d12984c68e).

The job cancellation approach is also based on the "How to cancel a coroutine" section of [this blog post](https://proandroiddev.com/android-coroutine-recipes-33467a4302e9).
