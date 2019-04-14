+++
title = "Threading"
weight = 20
+++

Kotlin coroutines are used for moving execution to background threads with ease. Instead of using coroutine primitives in our code arbitrarily, we'll use methods provided by the architecture in specific layers. There are two dispatchers that the framework prescribes, the UI (a wrapper for [`Dispatchers.Main`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-main.html)) and IO (a wrapper for [`Dispatchers.IO`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-i-o.html)) dispatchers.

First, we'll always launch every task as a coroutine with the UI dispatcher from our ViewModels with the `execute` method defined in the `JobViewModel` base class. This extends the previously discussed `RainbowCakeViewModel`, and takes the same type parameter, the view state:

```kotlin
class UserViewModel @Inject constructor(
        private val userPresenter: UserPresenter
): JobViewModel<UserViewState>(Loading) {

    fun updateUser() = execute {
        val name = userPresenter.getUserName()      // suspends execution on the UI thread, goes to background
        viewState = viewState.copy(userName = name) // resumes execution on the UI thread again
    }

}
```

(Notice that the presenter we're calling here is injected via constructor injection with Dagger.)

Then we make a single switch from the UI dispatcher to the IO dispatcher whenever a presenter function is called. The code inside `withIOContext` will be run on a background thread pool:

```kotlin
class UserPresenter @Inject constructor() {
    suspend fun getUserName(): String = withIOContext { // contents are executed on the IO threadpool
        "Jane" + " " + "Doe" // this StringBuilder is allocated and used on the background thread
    }
}
```

This ensures that any code below the presenters runs on background threads. If needed, additional dispatcher changes can be performed in lower layers. Read more [here](/content/concepts/threading.md).

![Architecture threading overview](/images/arch_threading.png)


## Retrofit and coroutines

Retrofit is a good example of a data source library that can be used with coroutines. You can either [directly declare suspending methods in your interfaces](https://zsmb.co/retrofit-meets-coroutines) since `2.5.1-SNAPSHOT`, or use a suspending call adapter, such as [this one](https://gist.github.com/zsmb13/c539cbce5ca9b85d9502436f2f286605) or [this one](https://github.com/JakeWharton/retrofit2-kotlin-coroutines-adapter). All of these approaches let us make our calls in a suspending way.

- The network call still has to block a thread at some point, but this won't be the UI thread, nor one of our IO threads. The API will suspend the coroutine on the IO thread we called it from and execute network calls on the default OkHttp threadpool (the same one that would've been used if we had a callback-based Retrofit API implementation with `enqueue`).
- These solutions support cancellation as well. If the coroutine is cancelled from the ViewModel, this cancellation will immediately run down the suspending call chain all the way to the data source, and cancel the network call directly and immediately. This means that if we leave a screen while a long network call is ongoing, the call won't still be completed in the background just to have its result thrown away when it's done, like with some other threading solutions.
