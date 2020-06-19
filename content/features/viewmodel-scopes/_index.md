+++
title = "ViewModel scopes"
weight = 20
+++

<div class="small-subtitle">rainbow-cake-dagger</div>
<div class="small-subtitle">rainbow-cake-koin</div>

ViewModels by default are scoped to their Fragment, meaning a new instance is created for every new instance of the Fragment (barring configuration changes), and they are cleared when their Fragment is destroyed without recreation.

There are use cases where it would make sense to share ViewModel instances between Fragments. Both the Dagger 2 and Koin integrations of RainbowCake provide support for this in their `getViewModelFromFactory` methods, in the form of the optional `scope` parameter.

>See the [Dependency Injection](/features/dependency-injection/) page for more details about these two integrations. 

_Note that these ViewModel scopes only exist in terms of Fragment ViewModels, as Activity ViewModels are always scoped to their Activity._

#### Default scope

The default behavior of ViewModels, produced by the simplest `provideViewModel` implementation.

```kotlin
override fun provideViewModel() = getViewModelFromFactory()
```

A ViewModel with this scope is scoped to its Fragment. A new ViewModel will be created each time a Fragment using this scope is instantiated, and it will be cleared when the Fragment is destroyed.

This scope can also be made explicit by providing it as a parameter:

```kotlin
override fun provideViewModel() = getViewModelFromFactory(scope = Default)
```

#### Parent Fragment scope

A ViewModel with this scope is scoped to the parent of its Fragment.

```kotlin
override fun provideViewModel() = getViewModelFromFactory(scope = ParentFragment)
```

A parent Fragment **must** exist in this case, otherwise instantiation will fail. ViewModels in this scope will be cleared when the parent Fragment is destroyed. The typical use case for this scope is multiple nested Fragments that need to share data amongst themselves through a ViewModel, e.g. those in a ViewPager.

A `key` may be provided to scope multiple separate instances of the same ViewModel class within the same parent Fragment.

```kotlin
override fun provideViewModel() = getViewModelFromFactory(scope = ParentFragment("key"))
```

#### Activity scope

A ViewModel with this scope is scoped to the current Activity.

```kotlin
override fun provideViewModel() = getViewModelFromFactory(scope = Activity)
```

ViewModels in this scope will only be cleared when this Activity is destroyed, meaning they might exist long after any Fragments using them have been destroyed.

A `key` may be provided to scope multiple separate instances of the same ViewModel class within the same Activity. 

```kotlin
override fun provideViewModel() = getViewModelFromFactory(scope = Activity("key"))
```

If no `key` is provided, the ViewModel will essentially exist as a singleton within the single Activity of the application. This is the recommended way for creating singleton ViewModel instances, as opposed to having them managed by Dagger, which takes the control away from the `ViewModelProvider` API, and can lead to unexpected behaviour (consider coroutine cancellation in this case!).

While not currently part of the architecture library, the following `typealias` may be used to make this intention more explicit: 

```kotlin
typealias Singleton = ViewModelScope.Activity

override fun provideViewModel() = getViewModelFromFactory(scope = Singleton)
```
