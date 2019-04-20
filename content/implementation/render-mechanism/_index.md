+++
title = "Render mechanism"
weight = 10
+++

The render mechanism is implemented in the `RainbowCakeViewModel` and `RainbowCakeFragment` classes. Here's the relevant part of `RainbowCakeViewModel`:

```kotlin
abstract class RainbowCakeViewModel<VS : Any>(initialState: VS) : ViewModel() {

    private val _state = MutableLiveData<VS>()

    init {
        _state.value = initialState
    }

    val state: LiveData<VS> = _state.distinct()

    protected var viewState: VS
        get() = _state.value!!
        set(value) {
            _state.value = value
        }

    // ...
}
```

- `_state` is a private backing property that holds the `MutableLiveData` wrapping the state. This is never accessed directly, other than inside the initializer block that sets it to `initialState` at construction time, which means that `_state` never holds a `null` value, it's always initialized to a valid state.
- `state` is a public property that exposes `_state` through the read-only `LiveData` interface for the `RainbowCakeFragment` to observe. It also calls the `distinct` extension on it, which prevents the `LiveData` from emitting the exact same state twice ([see here](/features/viewstate/#distinct-states-only-equalsmatters)).
- `viewState` is a convenience property that lets subclasses of `RainbowCakeViewModel` read and write the current state without having to know that it's stored in a `LiveData`.

This state is then trivially observed in the `RainbowCakeFragment`'s `onViewCreated` method, and delegated to the `render` method that subclasses must override:

```kotlin
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)

    viewModel.state.observe(viewLifecycleOwner, Observer { viewState ->
        viewState?.let { render(it) }
    })
    // ...
}
```


### Lifecycles

Fragments have two lifecycles that could be used to observe the `LiveData` in the ViewModel.

#### Fragment lifecycle

One of them is the lifecycle of the Fragment itself, which starts when the Fragment is created, and ends when it is destroyed. This starts with the `onCreate` method, and ends with `onDestroy`. To observe `LiveData` with this lifecycle, the observation would have to start in `onCreate`, and the Fragment instance itself would be passed in as a parameter.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    viewModel.state.observe(this, Observer { ... })
}
``` 

The issue with using this lifecycle is that a Fragment's view may be recreated any number of times during its lifetime, and we have to populate that new view with data again. However, our observer won't be triggered after the creation of a new view, as `LiveData` does not notify observers if the data they're observing hasn't changed. This would leave any new views uninitialized!

We could attempt to fix this by starting observation in the `onViewCreated` method instead, like so:

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    viewModel.state.observe(this, Observer { ... })
}
```

Since we'd be creating a new `Observer` instance each time the Fragment gets a new view, each of them would be invoked at least once, therefore properly populating our view. However, if we use the Fragment itself as the `LifecycleOwner`, these observations will only be cleared up when the Fragment is destroyed. Until then, with every new view created, a new `Observer` will be attached to the `LiveData`, and when the state does change, all of them will be triggered one after another, running the `render` method multiple times. 

#### View lifecycle

This shorter lifecycle of the Fragment's view is also available to use with our observations. This can run its course multiple times over the lifecycle of a Fragment. It starts with `onCreateView`/`onViewCreated`, and it ends with `onDestroyView`. This is the lifecycle actually used to observe view state. We make the `observe` call in the `onViewCreated` method, but now use `viewLifecycleOwner` as its parameter. This owner's lifecycle will end in `onDestroyView`, automatically cleaning up the observation.

```kotlin
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)

    viewModel.state.observe(viewLifecycleOwner, Observer { ... })
    // ...
}
``` 

These issues are also discussed in [this article](https://medium.com/@BladeCoder/architecture-components-pitfalls-part-1-9300dd969808) and in [this issue](https://github.com/googlesamples/android-architecture-components/issues/47).
