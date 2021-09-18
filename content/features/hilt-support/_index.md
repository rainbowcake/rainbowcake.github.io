+++
title = "Hilt support"
weight = 20
+++

<div class="small-subtitle">rainbow-cake-dagger</div>

RainbowCake's primary dependency injection mechanism remains Dagger 2, as shown by other pages of this documentation that discuss DI. However, it also ships with an artifact that gives you simple [Hilt](https://dagger.dev/hilt/) support for your DI needs.

**For a starter project that already has Hilt set up for use with RainbowCake, check out [Blank-Hilt](https://github.com/rainbowcake/sample-blank-hilt).**

> See the [Dependency Injection](/features/dependency-injection/) page for more details about dependency injection in RainbowCake in general. 

### Setup

To get started with RainbowCake and Hilt, include the `rainbow-cake-hilt` artifact, as described in detail on the [Dependencies](/getting-started/dependencies/) page.

You'll also have to set up Hilt in your project, following the steps in the [official Android guide](https://developer.android.com/training/dependency-injection/hilt-android).

This includes:
- Adding the Gradle plugin
- Adding the dependency
- Annotating your application with `@HiltAndroidApp`
- Annotating your Activity with `@AndroidEntryPoint`

### Per-screen usage

Add the `@HiltViewModel` annotation to your RainbowCake ViewModels:

```kotlin
@HiltViewModel
class UserViewModel @Inject constructor(
    private val userPresenter: UserPresenter
) : RainbowCakeViewModel<UserViewState>(Loading) {
    // ...
}
```

Annotate each individual screen with `@AndroidEntryPoint`, and then use the `getViewModelFromFactory` method from the `hilt` package:

```kotlin
import co.zsmb.rainbowcake.hilt.getViewModelFromFactory

@AndroidEntryPoint
class UserFragment : RainbowCakeFragment<UserViewState, UserViewModel>() {
    override fun provideViewModel() = getViewModelFromFactory()
}
```

That's it! Now your RainbowCake app is powered by Hilt.
