+++
title = "ViewFlipper"
weight = 20
+++

### Introduction

As seen in the [tutorial's example](/getting-started/tutorial/state-handling/), when you have a screen with completely distinct states, you can end up with code in each branch of your `render` method that hides or displays Views. Here's a refresher of what that code looked like:

```kotlin
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
```  

This code can grow even more if there are more than just two states, and the rendering code for each state has to first hide the possibly visible Views from all other states. The [`ViewFlipper`](https://developer.android.com/reference/android/widget/ViewFlipper) component can be a helpful tool in your toolbox in this case. `ViewFlipper` is essentially a fancy `FrameLayout` which allows you to easily display only a single child inside it at a time.

For the user screen, you might use it in your layout like this:

```xml
<ViewFlipper xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/viewFlipper"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ProgressBar
        android:layout_width="64dp"
        android:layout_height="64dp"
        android:layout_gravity="center" />

    <ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <!-- Profile image, TextViews, etc -->

    </ConstraintLayout>

</ViewFlipper>
```

Then, your `render` method need only flip to the appropriate page index according to the current state, which is just a single line of code:

```kotlin
override fun render(viewState: UserViewState) {
    when (viewState) {
        is Loading -> {
            viewFlipper.displayedChild = 0
        }
        is UserLoaded -> {
            viewFlipper.displayedChild = 1
            
            usernameText.text = viewState.userName
            Glide.with(profileImage).load(viewState.profileImage).into(profileImage)
        }
    }.exhaustive          
}
```  

### Readability and organization

There are two further improvements to be made to improve this usage of a `ViewFlipper`.

The first of this concerns the layout XML. If the views of your states get complex, you'll end up with a single, long XML file to edit, which is hard to find your way around. Furthermore, only a `ViewFlipper`'s first child shows up in layout previews, which makes editing any of the other layouts very tedious.

To combat this, it's recommended that any non-trivial layouts are extracted into separate files, and added with `include` tags in the `ViewFlipper` instead:

```xml
<ViewFlipper xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/viewFlipper"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <!-- 0 -->
    <ProgressBar
        android:layout_width="64dp"
        android:layout_height="64dp"
        android:layout_gravity="center" />

    <!-- 1 -->
    <include layout="@layout/layout_user_loaded" />

</ViewFlipper>
```

The second improvement is for your Kotlin code. Paging to a `0` or `1` index might make sense now, but these are the sort of magic numbers that can get very confusing with time. It's good practice to introduce well-named constants for these. These can be placed in the `companion object`, or in another nested `object`:

```kotlin
private object Flipper {
    const val LOADING = 0
    const val USER_LOADED = 1
}

override fun render(viewState: UserViewState) {
    when (viewState) {
        is Loading -> {
            viewFlipper.displayedChild = Flipper.LOADING
        }
        is UserLoaded -> {
            viewFlipper.displayedChild = Flipper.USER_LOADED
            
            usernameText.text = viewState.userName
            Glide.with(profileImage).load(viewState.profileImage).into(profileImage)
        }
    }.exhaustive          
}
```

### Notes

>For some more tips about rendering single view states, take a look at [Designing and Working with Single View States on Android](https://zsmb.co/designing-and-working-with-single-view-states-on-android/). 
