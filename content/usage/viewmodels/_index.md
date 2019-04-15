+++
title = "ViewModels"
weight = 20
+++

##  About `execute`

The `execute` method from `JobViewModel` is used to launch a coroutine on the UI thread with the appropriate `CoroutineContext` in ViewModels.

A basic example of this:

```kotlin
class UserViewModel @Inject constructor(
        private val userPresenter: UserPresenter
) : JobViewModel<UserViewState>(Loading) {
    
    fun loadUser(id: String) = execute {
        // This code passed to `execute` is in a coroutine, and can call
        // suspending methods of the Presenter. It can also update the
        // viewState, as it's on the UI thread at this level.
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
launch(Contexts.UI + rootJob) {
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

In the real implementation, you also get the option to disable this behaviour, and run multiple `execute` code freely in parallel. You just have to pass `false` for the `blocking` argument of the `execute` method:

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

### ViewModel error handling

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

This behaviour can be modified by changing the framework's configuration, as described [here](/usage/configuration/).

## Dagger integration for ViewModels

ViewModels are injected into Fragments via Dagger. For the injection mechanism to know about a given ViewModel, it has to be declared in a Dagger module, conventionally named `ViewModelModule`:

```kotlin
@Module
abstract class ViewModelModule {
    @Binds
    @IntoMap
    @ViewModelKey(MainViewModel::class)
    abstract fun bindMainViewModel(mainViewModel: MainViewModel): ViewModel
    
    @Binds
    @IntoMap
    @ViewModelKey(UserViewModel::class)
    abstract fun bindUserViewModel(userViewModel: UserViewModel): ViewModel

    // ...
}
```

Each binding requires an abstract function that takes the given specific ViewModel as a parameter, and is annotated with `@ViewModelKey`, with the same ViewModel class specified in the annotation as the key. Their return type always has to be the base `ViewModel` type. This is called a *multibinding*, see the [references](#references) to learn more about them.

## Data fetch choices

While ViewModels are able to hold their state through various configuration changes while the screen is still active, it still has to be decided when this state is loaded and reloaded. Here are two common options you may want to use:

- Reload state every time the Fragment attached to the `ViewModel` becomes visible. The convention for doing this is to define a `load` method in the ViewModel (this might take parameters, such as the `Fragment`'s arguments), and call this in the `Fragment`'s `onStart` method. This is practical when you want the `Fragment` to show up-to-date state even when it becomes visible after navigating back to it from another screen, which may have modified data it needs to displays.
- Load state once and hold onto it for the entire lifetime of the `Fragment` (of course, updates triggered by user input may still modify the state). This is best done by fetching the state in the `ViewModel`'s initializer block (`init`). This approach is useful if fetching the data to display is an expensive operation, or would always fetch the same data anyway.
