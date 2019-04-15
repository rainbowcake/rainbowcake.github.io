+++
title = "State handling"
weight = 20
+++

Each screen (or view) in the application is a Fragment. (Activity based screens are also technically supported, but this documentation will focus on Fragments, as that's the recommended approach.) The `RainbowCakeFragment` class provides integration with dependency injection to grab the appropriate ViewModel for a given Fragment. It also connects to and disconnects from the ViewModel automatically, and receives state changes and events.

View states may be created in one of two ways, either as [sealed classes](/usage/viewstate/#sealed-class-view-states) or just [data classes](/usage/viewstate/#data-class-view-state-implementations). Most of the time, the former of these is necessary - the latter will only work for very simple screens.

For this section, we'll look at a simple screen that loads a user profile, displaying a username and profile image. Here's the sealed class representing our view state:

```kotlin
sealed class UserViewState

object Loading : UserViewState()

data class UserLoaded(
        val userName: String,
        val profileImage: Uri
) : UserViewState()
```

The `UserViewModel` class inherits from `RainbowCakeViewModel`, and it provides the base class with its view state class as a type parameter, as well as its initial state via the constructor call. When the state of the view has to be updated, we use the `viewState` property of `RainbowCakeViewModel` to set the new one:

```kotlin
class UserViewModel : RainbowCakeViewModel<UserViewState>(Loading) {
    fun loadUser() {
        viewState = UserViewState(userName = "Jane Doe")
    }
}
```

Now, let's see the `UserFragment` implementation. It will inherit from `RainbowCakeFragment`, and pass both the view state and ViewModel as type parameters to it. It will also implement three required methods:
- `provideViewModel` must always call the `getViewModelFromFactory()` helper function provided with the base classes.
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
        }          
    }
    
    //...
}
```
