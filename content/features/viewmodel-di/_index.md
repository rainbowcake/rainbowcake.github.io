+++
title = "ViewModel DI"
weight = 20
+++

<div class="small-subtitle">rainbowcake-core</div>

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

Each binding requires an abstract function that takes the given specific ViewModel as a parameter, and is annotated with `@ViewModelKey`, with the same ViewModel class specified in the annotation as the key. Their return type always has to be the base `ViewModel` type.

To learn more about this setup, see the [implementation details](/implementation/viewmodels/).
