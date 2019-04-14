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
