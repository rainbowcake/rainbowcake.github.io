+++
title = "Presentation"
weight = 30
+++

This is how our ViewModel currently looks:

```kotlin
class UserViewModel : RainbowCakeViewModel<UserViewState>(Loading) {
    fun loadUser() {
        viewState = UserViewState(userName = "Jane Doe", profileImage = null)
    }
}
```

A hardcoded user is rarely the data we want to display. This user will probably come from a database or API call in our application. It's time to build out some of the lower layers of our architecture.

The next layer will be the the presenter. We'll be injecting this class into our ViewModel using Dagger, like so: 

```kotlin
class UserViewModel @Inject constructor(
    private val userPresenter: UserPresenter
): RainbowCakeViewModel<UserViewState>(Loading) {
    // ...
}
```

As described in the [overview](/#overview), something important needs to happen when we make the jump from the ViewModel layer to the Presenter. This is where our call is put on a background thread. Our tool for this is Kotlin coroutines.

Instead of using coroutine primitives in our code arbitrarily, we'll use methods provided by the architecture. There are two dispatchers that the framework prescribes, the UI (a wrapper for [`Dispatchers.Main`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-main.html)) and IO (a wrapper for [`Dispatchers.IO`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-i-o.html)) dispatchers.

First, we'll always launch every task as a coroutine with the UI dispatcher from our ViewModel with the `execute` method defined in the `JobViewModel` base class. This extends the previously discussed `RainbowCakeViewModel`, and takes the same type parameter, the view state:

```kotlin
class UserViewModel @Inject constructor(
        private val userPresenter: UserPresenter
): JobViewModel<UserViewState>(Loading) {

    fun loadUser() = execute {
         // We set the Loading state while we're fetching data
         viewState = Loading
         
         // getUser is a suspending call, which lets go of the
         // UI thread while it gets us the required data
         val user = userPresenter.getUser()
         
         // Back on the UI thread, put data in view state
         viewState = UserViewState(userName = user.name, profileImage = user.image)
     }

}
```

In order for `getUser` to suspend the UI thread, we make a single switch from the UI dispatcher to the IO dispatcher when it's called. We'll do this in every public presenter method. The helper function provided by the architecture for this is the `withIOContext`  method. Any code inside it will be run on a background thread pool:

```kotlin
class UserPresenter @Inject constructor() {
    suspend fun getUser(): User = withIOContext { 
        // the contents of withIOContext are executed on the IO threadpool
        // this User object is allocated on a background thread
        User("Jim", "https://some.domain/images/34")
    }
    
    // ...
}
```

### A note about lifecycles

Tasks launched via the `RainbowCakeViewModel`'s `execute` method survive configuration changes since they're tied to the ViewModel instance, which also survives said changes. This means that for example, a network request may be fired off for one instance of a Fragment, and it will continue execution even if the Fragment is recreated. Regardless of the Fragment's lifecycle changes, the result of the call will arrive in the ViewModel's view state, and whenever a Fragment instance binds to the ViewModel, it will receive the results to display. 

View state storage is implemented behind the scenes using LiveData. This provides the benefit of no references being kept to inactive views by the lower layers. If a result arrives for a Fragment that no longer exists, by default it will be placed in the ViewModel, which is simply never observed by anyone, and is eventually garbage collected. This way, inactive UI elements are never updated.

There is, however, an even stronger [cancellation mechanism](/implementation/job-cancellation/) in place for tasks for which ignoring the result isn't enough, meaning they shouldn't even continue execution if their respective screen is closed. Every coroutine launched by `execute` calls will be cancelled if the ViewModel is cleared (its view is permanently destroyed). Lower level calls that want to be cancellable by this mechanism have to use suspending functions and support cooperative coroutine cancellation properly.
