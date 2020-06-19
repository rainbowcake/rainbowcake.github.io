+++
title = "View state"
weight = 20
+++

<div class="small-subtitle">rainbow-cake-core</div>

>This page is just a brief comparison of sealed class and data class based view states. For some more tips about using single view states, have a look at [Designing and Working with Single View States on Android](https://zsmb.co/designing-and-working-with-single-view-states-on-android/).

###  Sealed class view states

The recommended style of view state to be used is based on [sealed classes](https://kotlinlang.org/docs/reference/sealed-classes.html). These are handy because they only have a known number of subclasses, and can therefore be checked exhaustively with `when` statements (see the docs linked above).

This style is to be used when a screen has multiple, completely distinct states which would be hard to describe with a single data class. For example, given a screen that can show a user's profile information, a loading progressbar, or a network error, you'd come up with a data class like this:

```kotlin
data class ProfileViewState(
        val errored: Boolean = false,
        val loading: Boolean = false,
        val name: String? = null
)
```

This would be problematic because it's possible to create completely nonsensical instances of this state. For example, you could do the following, which doesn't really make any sense - which of its three states would the screen even show?

```kotlin
viewState = ProfileViewState(errored = true, loading = true, name = "Sally")
```

The same screen approached with sealed classes looks like this:

```kotlin
sealed class ProfileViewState

object Errored : ProfileViewState()

object Loading : ProfileViewState()

data class ProfileLoaded(val name: String) : ProfileViewState()
```

This clearly divides these states so that there's only a single, valid one stored in the ViewModel, and the Fragment's rendering logic is straightforward from here:

```kotlin
override fun render(viewState: ProfileViewState) {
    errorLayout.isVisible = false
    profileLayout.isVisible = false
    loadingLayout.isVisible = false
    
    when (viewState) {
        is Loading -> {
            loadingLayout.isVisible = true
        }
        is Errored -> {
            errorLayout.isVisible = true
        }
        is ProfileLoaded -> {
            profileLayout.isVisible = true
            tvName.text = viewState.name
        }
    }
}
```

Note how the `name` property in the last branch is available on the `viewState` parameter, because it's already known to be of type `ProfileLoaded` after passing the type check of the `when` statement.

### Data class view state implementations

The simpler style of view state is a single data class. The power within this approach comes from the data class' generated `copy` method, which allows us to make partial, incremental updates to the current state of a view.

Let's take the following example:

```kotlin
data class CartViewState(
        val cartItems: List<CartItem> = emptyList(),
        val canProceedToOrder: Boolean = false
)
```

Our screen that has this ViewState displays the list of cart items, and has a button that lets the user complete their current order. The `canProceedToOrder` variable can have arbitrary business logic behind it, let's say it will be `true` when the user has accepted the terms and conditions of the application, with a checkbox on the screen.

Initially, when declaring the ViewModel, you can create a `CartViewState` with all default parameters:

```kotlin
class CartViewModel @Inject constructor(
        private val cartPresenter: CartPresenter
) : RainbowCakeViewModel<CartViewState>(CartViewState()) { /* ... */ }
```

Later, you can load the cart items so that they are displayed on the screen. On the first load, the `canProceedToOrder` property won't have a set value anyway, so you can create an entirely new ViewState to replace the old one. Here, the default `false` value of the property is being used again.

```kotlin
viewState = CartViewState(cartPresenter.getCartItems())
```

When the user clicks the checkbox to accept the terms, however, you definitely don't need to load the cart items again, so a partial update of the current ViewState is enough.

The obvious solution would be this one:

```kotlin
viewState = CartViewState(cartItems = viewState.cartItems, canProceedToOrder = true)
```

This is quite verbose even with just one unchanged property though. If you had 5 unchanged properties in the ViewState, you'd have to read all of their values and pass them into the new ViewState instance.

The same can be achieved using the aforementioned `copy` method in a much cleaner way. This operation keeps any existing property values of the previous ViewState, and only modifies the ones you pass into it (named parameters are *highly* recommended here for legibility).

```kotlin
viewState = viewState.copy(canProceedToOrder = true)
```

### Distinct states only (#equalsmatters)

`RainbowCakeViewModel` contains logic that dispatches only "new" states to the Fragment attached to it. Setting the same state twice in a ViewModel won't trigger two `render` calls in the Fragment, just one - since this should make no difference in the result on the view anyway.

How are states compared? By their [`equals`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-any/equals.html) implementations, which is what's accessed in Kotlin with the `==` operator. This makes it important that all classes that act as states should be:

- *data classes*, which get automatic `equals` implementations that compare all of their primary constructor parameters, or
- *objects*, which only have one instance and hold state, therefore `equals` comparisons work on them by default.

Having a ViewState class with an improper `equals` implementation shouldn't cause any major issues, but it will hurt performance by making their Fragment occasionally render the same state multiple times.
