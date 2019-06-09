+++
title = "State handling"
weight = 10
+++

_**Note:** you should've read the [overview on the home page](/#overview) before jumping into this material._

Each screen (or view) in the application is a Fragment. While Activity based screens are also supported, most of the documentation will focus on Fragments, as that's the recommended approach. 

The `RainbowCakeFragment` base class provides integration with dependency injection to grab the appropriate ViewModel for a given Fragment. It also connects to and disconnects from the ViewModel automatically, and receives state changes and events.

View states may be created in one of two ways, either as [sealed classes](/features/viewstate/#sealed-class-view-states) or just [data classes](/features/viewstate/#data-class-view-state-implementations). Most of the time, the former of these is necessary - the latter will only work for very simple screens.

_The [screen template](https://github.com/rainbowcake/rainbowcake-templates#screen-template) for the architecture can generate both styles for you._

For now, we'll look at a simple screen that loads a user profile, displaying a username and profile image. Here's the sealed class representing our view state:

```kotlin
sealed class UserViewState

object Loading : UserViewState()

data class UserLoaded(
        val userName: String,
        val profileImage: Uri? = null
) : UserViewState()
```

We'll need a ViewModel for our screen. This `UserViewModel` class inherits from `RainbowCakeViewModel`, and it provides the base class with its view state class as a type parameter, as well as its initial state (`Loading`) via the constructor call. When the state of the view has to be updated, we use the `viewState` property of `RainbowCakeViewModel` to set the new one:

```kotlin
class UserViewModel : RainbowCakeViewModel<UserViewState>(Loading) {
    fun loadUser() {
        viewState = UserLoaded(userName = "Jane Doe")
    }
}
```

Now, let's see the `UserFragment` implementation. It will inherit from `RainbowCakeFragment`, and pass both the view state and ViewModel as type parameters to it. It will also implement three required methods:

- `provideViewModel` must always call the `getViewModelFromFactory()` helper function that's provided by the architecture library.
- `getViewResource` should return the layout XML to be inflated for the Fragment.
- `render` is how the Fragment observes the state stored in the ViewModel, and is called every time the view state changes. Its responsibility is to update the state of the UI to reflect the current view state. It must be implemented in a way so that any previous view states do not affect the current state of the UI. In other words, the same view state being set must always result in the same state for the Fragment's UI.

```kotlin
class UserFragment : RainbowCakeFragment<UserViewState, UserViewModel>() {
    override fun provideViewModel() = getViewModelFromFactory()
    override fun getViewResource() = R.layout.fragment_user

    override fun render(viewState: UserViewState) {
        when (viewState) {
            is Loading -> {
                progressBar.isVisible = true
                profileContainer.isVisible = false
            }
            is UserLoaded -> {
                progressBar.isVisible = false
                profileContainer.isVisible = true
                
                usernameText.text = viewState.userName
                Glide.with(profileImage).load(viewState.profileImage).into(profileImage)
            }
        }.exhaustive          
    }
    
    //...
}
```

Note the use of the `exhaustive` extension property. This no-op property forces our `when` clause to be exhaustive, so that we are forced to handle all possible states of our screen. This is made possible by using a [sealed class](https://kotlinlang.org/docs/reference/sealed-classes.html) for our view state.

In order to move from the `Loading` to the `UserLoaded` state, our Fragment has to call the ViewModel's `loadUser` method at some point. There are [multiple good choices for doing this](/features/viewmodels/#data-fetch-choices), one of them is in the `onStart` method:

```kotlin
class UserFragment : RainbowCakeFragment<UserViewState, UserViewModel>() {
    
    //...
    
    override fun onStart() {
        super.onStart()
        viewModel.loadUser()
    }
}
```

This will trigger a change in the view state, and run the `render` method, displaying the user. Yay!
