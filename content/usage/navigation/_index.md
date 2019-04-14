+++
title = "Navigation"
weight = 20
+++

## Basics

Navigation is one of the only responsibilities of the single Activity that the application uses. Its layout is a single `FrameLayout` for Fragments to fill with content, and it implements a `Navigator` interface that performs the appropriate `FragmentManager` calls.

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

## Available methods

TODO

## Animations

TODO
