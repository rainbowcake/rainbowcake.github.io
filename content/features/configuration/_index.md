+++
title = "Configuration"
weight = 20
+++

<div class="small-subtitle">rainbow-cake-core</div>

The framework has a configuration DSL, which can be invoked in the `onCreate` method of your `Application` class.

Its usage looks like the following:

```kotlin
override fun onCreate() {
    super.onCreate()

    rainbowCake {
        isDebug = false
        logger = Loggers.NONE
        consumeExecuteExceptions = true
    }    
}
```

The available settings, and their possible values:

- `isDebug`: Boolean, `false` by default.
    - If set to false, it disables all internal logging of the framework, regardless of the setting of `logger`. May affect other behaviour in the future as well (in debug mode, prod behaviour will definitely not change). Recommended value is `BuildConfig.DEBUG`.
- `consumeExecuteExceptions`: Boolean, `true` by default (to keep existing behaviour).
    - Determines whether the `execute` method in `JobViewModel` should catch and log any uncaught exceptions in coroutines, or let them crash the app. Recommended to be set to `false` at the very least for debug builds, and should be considered even for production.
- `logger`
    - Determines how the framework should log its internal events. Available options by default are `Loggers.NONE` (as in no logging) and `Loggers.ANDROID` (logs to Logcat via `Log.d`).
    - If the `rainbowcake-timber` dependency is included, `Loggers.TIMBER` may also be used to log via Timber. Note that this doesn't `plant` any `Tree`s, you still have to do that yourself.

