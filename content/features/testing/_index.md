+++
title = "Testing"
weight = 20
+++

<div class="small-subtitle">rainbow-cake-test</div>

The architecture ships with a dedicated testing module, which supports unit testing the architecture. This testing module is in an experimental status, as it itself relies on experimental coroutine testing libraries at this point.

### Presenter testing

For Presenter tests, you can use the `PresenterTest` base class, which will replace the IO dispatcher used in Presenters with the test dispatcher, to make it execute anything scheduled on it immediately.

### ViewModel testing

For ViewModel tests, you can use the `ViewModelTest` base class, which will replace the Main dispatcher used by the `execute` method with the test dispatcher, and also replace the internal `LiveData` executor with a mock executor that executes everything on a single thread, in a blocking manner.

The difficult part of testing ViewModels is having to observe their reactions to inputs through various `LiveData`-based mechanisms - both state changes and events work this way. To make this easy, the `rainbow-cake-test` library provides the `observeStateAndEvents` function that lets you assert changes to state, as well as any emitted events:

```kotlin
vm.observeStateAndEvents { stateObserver, eventsObserver ->
    vm.loadArticle(1L)
    stateObserver.assertObserved(Loading, ArticleLoaded())
    vm.loadArticle(-1L)
    eventsObserver.assertObserved(InvalidIdError)
}
```

See the extensions on the `MockObserver` class for the currently available assertions. Note that you can also add your own assertion extensions on this class, as needed.

### All other tests

Testing other, lower level components such as Interactors or Data Sources should not require additional support from the architecture. You can use the experimental [coroutines test library](https://github.com/Kotlin/kotlinx.coroutines/tree/master/kotlinx-coroutines-test) to wrap such tests in `runBlockingTest` calls.
