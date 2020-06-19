+++
title = "Navigation"
weight = 20
+++

<div class="small-subtitle">rainbow-cake-navigation</div>

>**Note:** `rainbow-cake-navigation` is an optional dependency of RainbowCake. Other navigation solutions can also be used in RainbowCake-based apps.

Navigation is one of the few responsibilities of the single Activity that the application uses (and sometimes, the only one). Its layout is a single `FrameLayout` for Fragments to fill with content, and it implements a `Navigator` interface that performs the appropriate `FragmentManager` calls.

```kotlin
interface Navigator {
    fun add(fragment: Fragment)
    fun replace(fragment: Fragment)
    fun pop(): Boolean
    // ...
}
```

Any Fragment can access the Activity's navigator implementation via the `navigator` extension property that's defined on the Fragment class. This means that inside a Fragment, adding a new Fragment to the navigation stack is as simple as:

```kotlin
navigator?.add(UploadFragment())
```

And closing the current screen looks like this:

```kotlin
navigator?.pop()
```

### Available methods

This stack, unlike the regular Fragment backstack, really is a stack of screens. The following methods can be used to manipulate it.

#### `add`

Adds a Fragment to the top of the stack, making it visible. This is the primary way to navigate forward to a new screen.

```kotlin
navigator?.add(SettingsFragment())
```

#### `replace`

Removes the Fragment currently on the top of the stack, and puts the new one in its place.

```kotlin
navigator?.replace(SettingsFragment())
```

#### `pop`

Removes the Fragment on top of the stack.

```kotlin
navigator?.pop()
```

#### `popUntil`

Removes Fragments until a Fragment with the given type is reached.

```kotlin
navigator?.popUntil(HomeFragment::class)
navigator?.popUntil<HomeFragment>()
```

#### `setStack(fragments)`

Empties the stack, and then places the provided Fragments on it.

```kotlin
navigator?.setStack(HomeFragment(), CategoryFragment(), ProductFragment())
navigator?.setStack(listOf(HomeFragment(), CategoryFragment(), ProductFragment()))
``` 

#### `closeApplication()`

Exits the entire application.

```kotlin
navigator?.closeApplication()
```

#### `executePending()`

Executes any queued up navigation actions immediately.

```kotlin
navigator?.run {
    popUntil(HomeFragment::class)
    add(CategoryFragment())
    executePending()
}
```

### Animations

The `Navigator` provided by the library contains a fade animation between screen changes by default. It's possible to override this default behaviour as needed.

You can override it globally, by providing new values for certain properties in your Activity that inherits from `NavActivity`:

```kotlin
class MainActivity : SimpleNavActivity() {

    override val defaultEnterAnim: Int = R.anim.slide_in_right
    override val defaultExitAnim: Int = R.anim.slide_out_left
    override val defaultPopEnterAnim: Int = R.anim.slide_in_left
    override val defaultPopExitAnim: Int = R.anim.slide_out_right
    
}
```

You can also override animations one by one, by using overloads of the `add` and `replace` methods:

```kotlin
navigator?.add(SomeFragment(),
    enterAnim = R.anim.slide_in_right,
    exitAnim = R.anim.slide_out_left,
    popEnterAnim = R.anim.slide_in_left,
    popExitAnim = R.anim.slide_out_right
)
```

Note that a simple `0` may be used for any of these values to disable an animation altogether.
