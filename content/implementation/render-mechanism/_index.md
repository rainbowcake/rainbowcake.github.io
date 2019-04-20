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
    viewModel = provideViewModel()

    viewModel.state.observe(viewLifecycleOwner, Observer { viewState ->
        viewState?.let { render(it) }
    })
    // ...
}
```


### References

TODO: why is the view lifecycle being used for LiveData observation

https://medium.com/@BladeCoder/architecture-components-pitfalls-part-1-9300dd969808

https://github.com/googlesamples/android-architecture-components/issues/47
