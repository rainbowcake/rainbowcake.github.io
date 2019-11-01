+++
title = "ViewModel execute"
weight = 20
+++

<div class="small-subtitle">rainbow-cake-core</div>

The `execute` method from `JobViewModel` is used to launch a coroutine on the UI thread with the appropriate `CoroutineScope` in ViewModels.

A basic example of this:

```kotlin
class UserViewModel @Inject constructor(
        private val userPresenter: UserPresenter
) : JobViewModel<UserViewState>(Loading) {
    
    fun loadUser(id: String) = execute {
        // This code passed to `execute` is in a coroutine, and can call
        // suspending methods of the Presenter. It can also update the viewState,
        // as it's on the UI thread at this level of the call stack.
        viewState = UserLoaded(userPresenter.getUser(id))
    }
    
}
```

### Blocking by default

By default, only one `execute` call can be in progress for a given ViewModel at a time. This behaviour is present for situations where asynchronous work is to be performed based on user input, and the UI is supposed to wait for the result, in the sense that it shouldn't launch additional jobs while the async work is ongoing.

*To put it simply, if you have a button that launches a 1 second disk write in the background and then updates the view, this prevents the user from pressing the button 5 times in a second, resulting in 5 concurrent disk writes and 5 view updates.*

The implementation does **not** make `execute` calls sequential - it just terminates additional `execute` calls if there's already one ongoing. 

*Again, to put it simply, it just throws away the additional 4 button presses by the user while the first disk write is still ongoing.*

A rather simplified version of the implementation of this blocking mechanism:

```kotlin
launch {
    if (busy) {
        Timber.d("Denying job launch, busy")
        return@launch
    }
    busy = true

    try {
        task()
    } finally {
        busy = false
    }
}
```

In the real implementation, you also get the option to disable this behaviour, and run multiple `execute` calls freely in parallel. You just have to pass `false` for the `blocking` argument of the `execute` method:

```kotlin
execute(blocking = false) {
    somePresenter.performWork()
}
```

### Variants

There are some variants of the regular `execute` method for special use cases.

#### `executeNonBlocking`

This is a convenience method that's exactly equivalent to calling `execute` with the `blocking = false` parameter.

#### `executeCancellable`

This method is the exact same as the `execute` method, except it returns the `Job` that it has started, which enables ViewModel code to manually cancel it, if necessary. The regular `execute` method has a `Unit` return value, since most of the time it's supposed to be used with an expression body ViewModel function, which would leak the `Job` to the Fragment by default:

```kotlin
fun load() = execute { // This method returns the result of `execute` to the Fragment! 
    // ...
} 
```
 
Since `executeCancellable` returns the `Job`, it should never be used in an expression body. Instead, write a regular function in these cases:

```kotlin
fun load() {
    // Set a property of the ViewModel, or do whatever you need the Job for
    job = executeCancellable {
        // ...
    }
}
```

### Error handling

Any exceptions that happen inside `execute` calls that aren't caught will be, by default, caught by the `execute` function. These are logged by default. Job cancellation exceptions (these happen when the `ViewModel` is cleared and the job is cancelled as a consequence) are logged separately.

```kotlin
try {
    task() // the lambda passed to `execute`
} catch (e: CancellationException) {
    log("Job cancelled exception:")
    log(e)
} catch (e: Exception) {
    log("Unhandled exception in ViewModel:")
    log(e)
}
```

This behaviour can be modified by changing the framework's configuration, as described [here](/features/configuration/).
