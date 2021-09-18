+++
title = "Dagger support"
weight = 20
+++

<div class="small-subtitle">rainbow-cake-dagger</div>

Dagger 2 is RainbowCake's primary dependency injection solution, via the `rainbow-cake-dagger` artifact which makes it easy to get started with the framework.

**For a starter project that already has Dagger 2 set up for use with RainbowCake, check out [Blank](https://github.com/rainbowcake/sample-blank).**

> See the [Dependency Injection](/features/dependency-injection/) page for more details about dependency injection in RainbowCake in general. 

> See the [Hilt Support](/features/hilt-support/) page if you're using Dagger with Hilt. 

### Setup

To get started with RainbowCake and Dagger, include the `rainbow-cake-dagger` artifact, as described in detail on the [Dependencies](/getting-started/dependencies/) page.

You'll also have to include Dagger in your dependencies:

```groovy
// Dagger
def dagger_version = '2.38.1'
implementation "com.google.dagger:dagger:$dagger_version"
kapt "com.google.dagger:dagger-compiler:$dagger_version"
```

> For the latest version see [the Dagger releases page](https://github.com/google/dagger/releases).
 
### Providing dependencies

For the injection mechanism to know about your ViewModels, they have to be declared in a Dagger module, conventionally named `ViewModelModule`, and placed in a `di` package within the app's root package:

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
}
```

Each binding requires an abstract function that takes the given specific ViewModel as a parameter, and is annotated with `@ViewModelKey`, with the same ViewModel class specified in the annotation as the key. Their return type always has to be the base `ViewModel` type.

>To learn more about this setup, see the [implementation details](/implementation/viewmodel-di-with-dagger/).

The constructors of your ViewModels should be annotated with `@Inject` so that Dagger knows how to create them:

```kotlin
class UserViewModel @Inject constructor(
    private val userPresenter: UserPresenter
): RainbowCakeViewModel<UserViewState>(Loading) {
    // ...
}
```

For the other classes that your ViewModels or lower layers depend on, you should also annotate their constructors with `@Inject`:

```kotlin
class UserPresenter @Inject constructor() {
    suspend fun getUser(): User = withIOContext { /* ... */ }
}
```

> This is just the very basics of a Dagger setup, you can also declare additional modules with dependencies, etc. See the [Dagger documentation](https://dagger.dev/) for more details.

### Component setup

RainbowCake's Dagger integration works with a single application level component with a `@Singleton` scope to create ViewModels.

This means that you'll have to create a Dagger component that contains `ViewModelModule` as well as the `RainbowCakeModule` that comes from the `rainbow-cake-dagger` artifact. Your component needs to extend `RainbowCakeComponent`:

```kotlin
@Singleton
@Component(
    modules = [
        RainbowCakeModule::class,
        ViewModelModule::class
    ]
)
interface AppComponent : RainbowCakeComponent
```

### Application class setup

Finally, you'll need an `Application` implementation class that extends `RainbowCakeApplication`. This will override the `injector` property with the type of the concrete Dagger component that will be used in the application, in this example, `AppComponent`. The `setupInjector` method also needs to be overridden appropriately - it should create and assign an instance of the component:

```kotlin
class MyApplication : RainbowCakeApplication() {

    override lateinit var injector: AppComponent

    override fun setupInjector() {
        injector = DaggerAppComponent.create()
    }

}
```
    
Make sure you set this application up in your `AndroidManifest.xml` file as well, in your `application` tag's `name` property.
