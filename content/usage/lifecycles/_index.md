+++
title = "Lifecycles"
weight = 20
+++

Tasks launched via the `RainbowCakeViewModel`'s `execute` method survive configuration changes since they're tied to the ViewModel instance, which also survives said changes. This means that for example, a network request may be fired off for one instance of a Fragment, and it will continue execution even if the Fragment is recreated. Regardless of the Fragment's lifecycle changes, the result of the call will arrive in the ViewModel's view state, and whenever a Fragment instance binds to the ViewModel, it will receive the results to display. 

Using LiveData for view state also provides the benefit of no references being kept to inactive views by the lower layers. If a result arrives for a Fragment that no longer exists, by default it will be placed in the ViewModel, which is simply never observed by anyone, and is eventually garbage collected. This way, inactive UI elements are never updated.

There is, however, an even stronger cancellation mechanism in place for tasks for which ignoring the result isn't enough, meaning they shouldn't even continue execution if their respective screen is closed. Every coroutine launched by `execute` calls will be cancelled if the ViewModel is cleared (its view is permanently destroyed). Lower level calls that want to be cancellable by this mechanism have to use suspending functions and support cooperative coroutine cancellation properly. Read more [here](/content/datasources#retrofit-and-coroutines).

