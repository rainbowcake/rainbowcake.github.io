+++
title = "Project setup"
weight = 5
+++

The [Blank](https://github.com/rainbowcake/sample-blank) demo project (or its [Koin variant](https://github.com/rainbowcake/sample-blank-koin)) is a great way to start a fresh project built on RainbowCake. If you choose to use one of these, you can skip this page of the tutorial!

However, if you don't wish to use the starter projects, here's what you need to set up.

### Screen one

You'll need a `ui` package that will contain your various screens. Having this specific name is required by the [screen template](https://github.com/rainbowcake/rainbowcake-templates#screen-template) which is the recommended way of creating new screens. After installing the template, create your first screen by invoking `New -> Other -> RainbowCake Screen` on the `ui` package, and following the steps.

Alternatively, see the [`ui/blank` package](https://github.com/rainbowcake/sample-blank/tree/master/app/src/main/java/com/example/blank/ui/blank) of the Blank project for the pieces of an empty screen setup.

### Dependency injection

You'll need the following three pieces of code in place for your basic DI setup with Dagger. These are usually placed in the `di` package.

> For a Koin powered setup, see the [Koin support](/features/koin-support/) page.

- A module that binds at least one ViewModel. By convention, as long as there's just one module containing ViewModels, it's named `ViewModelModule`:

    ```kotlin
    @Module
    abstract class ViewModelModule {
        @Binds
        @IntoMap
        @ViewModelKey(UserViewModel::class)
        abstract fun bindUserViewModel(userViewModel: UserViewModel): ViewModel
    }
    ```

- A component that contains this module as well as the `RainbowCakeModule` that comes from the library artifact. This component also needs to extend `RainbowCakeComponent`:

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

- Finally, an Application class that extends `RainbowCakeApplication`. This will override the `injector` property with the type of the concrete Dagger component that will be used in the application, in this example, `AppComponent`. The `setupInjector` method also needs to be overridden appropriately - it should create and assign an instance of the component:

    ```kotlin
    class MyApplication : RainbowCakeApplication() {
    
        override lateinit var injector: AppComponent
    
        override fun setupInjector() {
            injector = DaggerAppComponent.create()
        }
    
    }
    ```
    
    Make sure you set this application up in your `AndroidManifest.xml` file as well, in your `application` tag's `name` property.
 

### Navigation

If you use `rainbowcake-navigation`, make your `Activity` inherit from `SimpleNavActivity`, and load your initial `Fragment` like so:

```kotlin
class MainActivity : SimpleNavActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        if (savedInstanceState == null) {
            navigator.add(UserFragment())
        }
    }

}
```
