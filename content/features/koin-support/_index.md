+++
title = "Koin support"
weight = 20
+++

<div class="small-subtitle">rainbow-cake-koin</div>

RainbowCake's primary dependency injection mechanism remains Dagger 2, as shown by other pages of this documentation that discuss DI. However, it also ships with an artifact that gives you simple [Koin](https://insert-koin.io/) support for your dependency injection needs.

**For a starter project that already has Koin set up for use with RainbowCake, check out [Blank-Koin](https://github.com/rainbowcake/sample-blank-koin).**

>See the [Dependency Injection](/features/dependency-injection/) page for more details about dependency injection in RainbowCake in general. 

### Setup

To get started with RainbowCake and Koin, include the `rainbow-cake-koin` artifact, as described in detail on the [Dependencies](/getting-started/dependencies/) page.

You'll also have to include Koin in your dependencies. The library comes in several different packages, here are some recommended ones to start with:

```groovy
def koin_version = '2.1.6'
implementation "org.koin:koin-core:$koin_version"
implementation "org.koin:koin-android:$koin_version"
implementation "org.koin:koin-android-viewmodel:$koin_version"
```

>For the latest version of Koin, see [its releases page](https://github.com/InsertKoinIO/koin/releases).

### Code differences

Instead of declaring a `ViewModelModule` class like in the Dagger setup, you can collect all of your ViewModels and Presenters in Koin modules, such as this:

```kotlin
val UIModule = module {
    factory { UserPresenter(get()) }
    factory { UserViewModel(get()) }
}
```

When using Koin, you won't need to create a component, and there is also no need to extend your application from a special base class. Instead, you will have to start up Koin within your application, declaring any modules that will supply your dependencies:

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        startKoin {
            androidContext(this@MyApplication)
            modules(UIModule)
        }
    }
}
```

Since they aren't being injected by Dagger, your classes no longer need `@Inject constructor`s. A simple ViewModel may look like this:

```kotlin
class UserViewModel(
    private val userPresenter: UserPresenter
): RainbowCakeViewModel<UserViewState>(Loading) {
    // ...
}
```

Finally, fetching a ViewModel in a Fragment happens the same way as with the Dagger setup, using the `getViewModelFromFactory` method, except it comes from a different package when using the Koin artifact:

```kotlin
import co.zsmb.rainbowcake.koin.getViewModelFromFactory

class UserFragment : RainbowCakeFragment<UserViewState, UserViewModel>() {
    override fun provideViewModel() = getViewModelFromFactory()
    // ...
}
```

That's it, you should be up and running with DI! For details on how to use Koin, take a look at [its official documentation](https://insert-koin.io/).
