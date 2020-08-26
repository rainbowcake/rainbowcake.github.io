+++
title = "Dependencies"
weight = 10
+++

The framework is available in several artifacts. This lets you pick and choose the features that you need in your project instead of having to pull in everything at once.

The currently available artifacts are listed below - you can get them all from MavenCentral.

```groovy
repositories {
    mavenCentral()
}
```

#### Core library

```groovy
implementation "co.zsmb:rainbow-cake-core:1.1.0"
```

The essentials of the architecture library: the view state mechanisms and the configuration entry points.

**This dependency is required to use RainbowCake.**

You will probably also want to include at least one of the dependency injection artifacts below, though you can also use your own DI solution. See the [Dependency Injection](/features/dependency-injection/) page for more details about DI in RainbowCake in general.

#### Dagger support

```groovy
implementation "co.zsmb:rainbow-cake-dagger:1.1.0"
```

The primary, recommended way of performing dependency injection when using RainbowCake, using Dagger 2. For details, see [Dagger support](/features/dagger-support/).

#### Koin support

```groovy
implementation "co.zsmb:rainbow-cake-koin:1.1.0"
```

An alternative dependency injection solution, powered by Koin 2.0. For details, see [Koin support](/features/koin-support/).

#### Testing utilities

```groovy
testImplementation "co.zsmb:rainbow-cake-test:1.1.0"
```

Testing helpers for RainbowCake projects. See the [Testing](/features/testing/) page for more details.

#### Navigation addon

```groovy
implementation "co.zsmb:rainbow-cake-navigation:1.1.0"
```

Navigation and argument handling features: `navigator`, `SimpleNavActivity`, and more. See [Navigation](/features/navigation/) for details.

>If you're using a solution such as the [Jetpack Navigation component](https://developer.android.com/guide/navigation/navigation-getting-started), you don't need this.

#### Channels addon

>**Note: the Channels artifact is DEPRECATED, and will not be present in future RainbowCake versions. To get continuous updates from lower layers, prefer using coroutine Flows.**

```groovy
implementation "co.zsmb:rainbow-cake-channels:0.7.0"
```

Version `0.7.0` of RainbowCake was the last version that shipped with this Channel support. If you use this, downgrade all other artifacts to `0.7.0` as well.

#### Timber addon

```groovy
implementation "co.zsmb:rainbow-cake-timber:1.1.0"
```

You only need this artifact if you want the framework to log about its internal events (exceptions caught by `RainbowCakeViewModel`, unhandled events), and you want it to do so using Timber. For more details, see [Configuration](/features/configuration/).
