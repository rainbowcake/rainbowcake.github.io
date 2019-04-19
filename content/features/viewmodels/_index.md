+++
title = "ViewModels"
weight = 20
+++

<div class="small-subtitle">rainbowcake-core</div>

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

Each binding requires an abstract function that takes the given specific ViewModel as a parameter, and is annotated with `@ViewModelKey`, with the same ViewModel class specified in the annotation as the key. Their return type always has to be the base `ViewModel` type. This is called a *multibinding*, see the [implementation details](/implementation/viewmodels/) to learn more about them.

## Data fetch choices

While ViewModels are able to hold their state through various configuration changes while the screen is still active, it still has to be decided when this state is loaded and reloaded. Here are two common options you may want to use:

- Reload state every time the Fragment attached to the `ViewModel` becomes visible. The convention for doing this is to define a `load` method in the ViewModel (this might take parameters, such as the `Fragment`'s arguments), and call this in the `Fragment`'s `onStart` method. This is practical when you want the `Fragment` to show up-to-date state even when it becomes visible after navigating back to it from another screen, which may have modified data it needs to displays.
- Load state once and hold onto it for the entire lifetime of the `Fragment` (of course, updates triggered by user input may still modify the state). This is best done by fetching the state in the `ViewModel`'s initializer block (`init`). This approach is useful if fetching the data to display is an expensive operation, or would always fetch the same data anyway.
