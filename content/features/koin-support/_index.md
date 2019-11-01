+++
title = "Koin support"
weight = 20
+++

<div class="small-subtitle">rainbow-cake-koin</div>

RainbowCake's primary dependency injection mechanism remains Dagger 2, as shown by every other page of this documentation that discusses DI. However, you may also choose to use Koin 2.0 for your dependency injection needs.

### Setup

To get started with RainbowCake and Koin, include the appropriate RainbowCake artifact, as described [here](/getting-started/setup/dependencies/).

You'll also have to include Koin in your dependencies. The library comes in several different packages, here are some recommended ones to start with:

```groovy
def koin_version = '2.0.1'
implementation "org.koin:koin-core:$koin_version"
implementation "org.koin:koin-android:$koin_version"
implementation "org.koin:koin-android-viewmodel:$koin_version"
```

### Code differences

Instead of declaring a `ViewModelModule` class, you can collect all of your ViewModels and Presenters in a Koin module, such as this:

```kotlin
val UIModule = module {
    factory { UserPresenter(get()) }
    factory { UserViewModel(get()) }
}
```

You no longer need a Dagger component, and there is no need to extend your application from a special base class. You will have to start up Koin within your application though:

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

Finally, fetching a ViewModel in a Fragment happens the same way as with the Dagger setup, using the `getViewModelFromFactory` method, except it comes from a different package.

```kotlin
import co.zsmb.rainbowcake.koin.getViewModelFromFactory

class UserFragment : RainbowCakeFragment<UserViewState, UserViewModel>() {
    override fun provideViewModel() = getViewModelFromFactory()
    // ...
}
```
