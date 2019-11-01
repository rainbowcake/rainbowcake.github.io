+++
title = "ViewModel DI"
weight = 10
+++

<div class="small-subtitle">rainbow-cake-dagger</div>

### One Factory to rule them all

`ViewModel`s can't be directly instantiated, as we need the framework to take care of their lifecycle aspect. The way to give a `ViewModel` a custom constructor is by using a custom `ViewModelProvider.Factory`. These factories look something like this, they receive the ViewModel class as a parameter when they need to instantiate one:

```kotlin
class MyViewModelFactory : ViewModelProvider.Factory {

    override fun <T : ViewModel?> create(modelClass: Class<T>): T {
        return  // ...
    }

} 
```

Inside this method, we have complete freedom to manufacture our `ViewModel` instance, so we'll make Dagger do it, gaining the ability to inject the `ViewModel`s the factory creates via their constructors so that they can just "ask for" dependencies there and not worry about where they come from. We can also use a single factory powered by Dagger which can instantiate any type of `ViewModel` that Dagger can provide. This is where the multibinding mechanism comes in.

Using multibindings, we construct a map in our `ViewModelModule`. The keys in the map are the `Class` values of each `ViewModel` (e.g. `UserViewModel::class`). The values in these map are the `ViewModel` *types* themselves (e.g. `UserViewModel`). 

This may seem redundant, but keep in mind that generally speaking, when using multibindings the keys we label our dependencies with could be other types as well, for example simply `String` or `Int`. We're only using `Class` as the key here because that's what the `ViewModelProvider.Factory` implementation will receive as a parameter when it has to instantiate a `ViewModel`. This will essentially let us ask Dagger at any time to create a ViewModel of a given `Class`.

```kotlin
@Module
abstract class RainbowCakeModule {

    @Binds
    abstract fun bindViewModelFactory(factory: ViewModelFactory)
        : ViewModelProvider.Factory

}
```

We've also bound the `DaggerViewModelFactory` we're implementing, so that we can inject the map we've constructed above into _it_ with Dagger. 

We could simply ask for a `Map<Class<ViewModel>, ViewModel>` in the `Factory`, but then we'd only get one concrete instance of `ViewModel` for each key (essentially, for each screen). If we opened the same screen multiple times, they'd have to have the same state because they'd receive the same `ViewModel` instances. This makes no sense (except for some rare cases).

Instead, we'll ask for the `Provider<ViewModel>` type as the map's values. A `Provider` is a class we can keep calling `get()` on, and each time it will manufacture new `ViewModel` instances, which will have all of its dependencies injected by Dagger appropriately.

This brings us to this implementation for our `ViewModelProvider.Factory`:
  
```kotlin
class DaggerViewModelFactory @Inject constructor(
        private val creators: Map<Class<ViewModel>, Provider<ViewModel>>
) : ViewModelProvider.Factory {

    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        return creators[modelClass].get() as T
    }

}
``` 

Let's review one more time:

- It receives a `Map` in its constructor from Dagger, which is backed by Dagger's injection mechanism to create any `ViewModel` that's been bound with its own `Class` as the key.
- Whenever we invoke `ViewModelProviders` and pass this `Factory` to it, it'll call the `Factory` with the concrete `ViewModel` `Class` we need to create.
- This `Factory` will fetch the appropriate `Provider` from the `Map` it stores in its `creators` property, and create a new instance of the required type of `ViewModel` by calling the `Provider`'s `get` method. This `ViewModel` will receive its dependencies via constructor injection, because it's instantiated by Dagger. 

Here's how these components are (roughly) connected:

![The Dagger powered ViewModel factory](/images/dagger_factory.png)

The code for `DaggerViewModelFactory` here is sample code that doesn't even quite compile, but I hope it gets the idea across. There's a couple more little things you need to take care of for this implementation to work in terms of generics, Kotlin-Java interop, and null handling. If you want to see the real, complete implementation, look at the `DaggerViewModelFactory` class of the framework.

### Getting the Factory in place

We still need to get an instance of this `Factory` into our `Fragment`s so that we can fetch `ViewModel`s with its help. 

We could add each of our `Fragment`s to our Dagger component as we're creating them, and inject them manually one by one, but this is tedious. It's a good idea to instead put the property that stores the `Factory` in the `RainbowCakeFragment` base class, but due to its type parameters, it practically can't be injected by Dagger. The workaround is to introduce another base class _above_ `RainbowCakeFragment` which is non-generic, and injects itself with the Factory:

```kotlin
abstract class InjectedFragment : Fragment() {

    @Inject
    lateinit var viewModelFactory: ViewModelProvider.Factory

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        injector.inject(this)
    }

}
``` 

This is no longer painful to add to our Dagger component, as it only has to be done once, and it injects itself neatly via the relatively simple field injection mechanism.

We could now fetch our `ViewModel` in our concrete `Fragment`s like this, and if it needs to be created, it will be created by Dagger somewhere in the `Factory`:

```kotlin
override fun provideViewModel(): ProfileViewModel {
    return ViewModelProviders.of(this, viewModelFactory)
                             .get(ProfileViewModel::class)
}
``` 

The `getViewModelFromFactory` method we actually use essentially performs this same call, with some extra handling for shared `ViewModel` scopes.


### References

For more about Dagger bindings and multibindings, see [here](https://proandroiddev.com/dagger-2-annotations-binds-contributesandroidinjector-a09e6a57758f) and [here](https://google.github.io/dagger/multibindings.html), for example.

The `ViewModelProvider.Factory` implementation is originally from a Google sample project, specifically, from their [GitHub Browser Sample](https://github.com/googlesamples/android-architecture-components/blob/17c315a050745c61ae8e79000bc0251305bd148b/GithubBrowserSample/app/src/main/java/com/android/example/github/viewmodel/GithubViewModelFactory.kt) app.
