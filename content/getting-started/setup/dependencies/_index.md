+++
title = "Dependencies"
weight = 10
+++

The framework is available in multiple artifacts - four, for now. This lets you pick and choose the features that you need in your project instead of having to pull in everything at once.

The currently available artifacts are listed below - you can get them all from MavenCentral.

```groovy
repositories {
    mavenCentral()
}
```

#### Core

```groovy
implementation "co.zsmb:rainbow-cake-core:0.4.0"
```

The essentials of the architecture library: the view state mechanisms and the configuration entry points. This dependency is required to use RainbowCake. You will also need to include at least one of the dependency injection artifacts below.

#### Dagger support

```groovy
implementation "co.zsmb:rainbow-cake-dagger:0.4.0"
```

The primary, recommended way of performing dependency injection when using RainbowCake, using Dagger 2.

#### Koin support

```groovy
implementation "co.zsmb:rainbow-cake-koin:0.4.0"
```

A new, alternative dependency injection solution, powered by Koin 2.0. For details, see [this page](/features/koin-support/).

#### Navigation addon

```groovy
implementation "co.zsmb:rainbow-cake-navigation:0.4.0"
```

All the navigation and argument handling features. `navigator`, `SimpleNavActivity`, and more.

#### Channels addon

```groovy
implementation "co.zsmb:rainbow-cake-channels:0.4.0"
```

Coroutine channel support in the form of - most notably - the `ChannelViewModel` base class and the `LiveData#toChannel` extension.

#### Timber addon

```groovy
implementation "co.zsmb:rainbow-cake-timber:0.4.0"
```

You only need this artifact if you want the framework to log about its internal events (this is mostly just the exceptions caught by `JobViewModel`), and you want it to do so using Timber. For more details, see [Configuration](/features/configuration/).
