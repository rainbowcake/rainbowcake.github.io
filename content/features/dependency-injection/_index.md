+++
title = "Dependency injection"
weight = 20
+++

RainbowCake ships with two different packages for DI integration, and you can also use your own completely custom solution with the framework. For the dependencies themselves, see [Dependencies](/getting-started/dependencies/).

### Dagger 2

The `rainbow-cake-dagger` artifact provides an easy way to get up and running using Dagger 2. This is the assumed default solution for DI in RainbowCake, and you'll see that integration being used in many pages of this documentation.

The setup instructions for the Dagger 2 support are described on the [Dagger support](/features/dagger-support/) page, while the [ViewModel DI with Dagger](/implementation/viewmodel-di-with-dagger/) page describes some of the underlying implementation details.

### Dagger Hilt

The `rainbow-cake-hilt` artifact provides an easy way to get up and running using Dagger 2. This is the assumed default solution for DI in RainbowCake, and you'll see that integration being used in many pages of this documentation.

The setup instructions for the Dagger 2 support are described on the [Dagger support](/features/dagger-support/) page, while the [ViewModel DI with Dagger](/implementation/viewmodel-di-with-dagger/) page describes some of the underlying implementation details.

### Koin

The [Koin](https://insert-koin.io/) support of the framework is provided in the `rainbow-cake-koin` module, and serves as a simple way of setting up RainbowCake projects with Koin. It's documented on its own separate page: [Koin support](/features/koin-support/).

### Integration point

The setup for dependencies and modules is different for the two provided (pun intended) packages, but with either of them, the point where you access your dependencies is the creation of the ViewModel.

This happens in your concrete implementation of a `RainbowCakeFragment`:

```kotlin
class ExampleFragment : RainbowCakeFragment<ExampleViewState, ExampleViewModel>() {
    override fun provideViewModel() = getViewModelFromFactory()
}
```

`provideViewModel` is an abstract method which you must override and return an instance of your ViewModel that belongs to your current Fragment instance. This must be of the same type as the second type argument passed to `RainbowCakeFragment` - your ViewModel type.

If you use `rainbow-cake-dagger` or `rainbow-cake-koin` packages, you'll get access to a `getViewModelFromFactory` method which makes the implementation a one-liner, and automagically grabs a ViewModel for you.

>If you want to customize the scoping of this ViewModel, check out the [ViewModel scopes](/features/viewmodel-scopes/) page.

### Custom DI (or none at all)

Since this is the only point where the DI system is accessed, you can also choose to use your own DI solutions. Even when using Dagger or Koin with RainbowCake, using the provided support for them is not mandatory, as you may want more control than those allow.

You can also omit using dependency injection altogether, and create ViewModels by calling the `ViewModelProvider` APIs directly (though in this case, you'd have to figure out how to connect your ViewModel to the lower layers of the architecture):

```kotlin
class ExampleFragment : RainbowCakeFragment<ExampleViewState, ExampleViewModel>() {

    override fun provideViewModel(): ExampleViewModel {
        return ViewModelProvider(this).get(ExampleViewModel::class.java)
    }

}
``` 
