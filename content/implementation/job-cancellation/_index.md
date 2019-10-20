+++
title = "Job cancellation"
weight = 10
+++

Internally, the `execute` method calls the `launch` coroutine builder to fire off a coroutine Job. The scope for these coroutines will be the ViewModel itself, as `JobViewModel` implements the `CoroutineScope` interface.

```kotlin
abstract class JobViewModel<VS : Any>(initialState: VS) 
    : RainbowCakeViewModel<VS>(initialState), CoroutineScope {

    private val rootJob: Job = SupervisorJob()

    final override val coroutineContext = Dispatchers.UI + rootJob
    
    // ...
}
```

The context of the scope is made up of two components: the UI dispatcher and a [`SupervisorJob`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-supervisor-job.html) instance that acts as the parent of all coroutines launched in the ViewModel.

- `Dispatchers.UI` describes the `CoroutineDispatcher` element to use, in this case, the Android UI thread until the job is placed on another dispatcher (for example, with a `withContext` call).
- `rootJob` provides the parent `Job` element for the coroutine. This is a simple, empty `Job`, only used to group the jobs launched inside a given ViewModel. The `rootJob` is cancelled  when the ViewModel's `onCleared` method is called - this is when the lifecycle it was attached to has terminated, meaning the `Activity`/`Fragment` is actually closed for good, and not just going through configuration changes.

```kotlin
override fun onCleared() {
    super.onCleared()
    rootJob.cancel()
    log("ViewModel cleared, rootJob cancelled")
}
```

Cancelling the `rootJob` will also cancel all of its child coroutines, which in this case is every coroutine launched with the `execute` method. Note that coroutine cancellation is cooperative. Blocking code running inside the coroutine will not be cancelled. Cancellation can only happen in suspending functions, and only if it's handled explicitly. [Retrofit's coroutine integration](/best-practices/retrofit-and-coroutines/) is a good example of this.

### References

The reasoning for using a `SupervisorJob` (as well as other good coroutine tips) is laid out nicely [in this article](https://proandroiddev.com/kotlin-coroutines-patterns-anti-patterns-f9d12984c68e).

The job cancellation approach is also based on the "How to cancel a coroutine" section of [this blog post](https://proandroiddev.com/android-coroutine-recipes-33467a4302e9).
