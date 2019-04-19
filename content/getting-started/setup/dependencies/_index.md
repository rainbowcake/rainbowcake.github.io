+++
title = "Dependencies"
weight = 10
+++

The framework is available in multiple artifacts - four, for now. This lets you pick and choose the features that you need in your project instead of having to pull in everything at once.

The currently available artifacts are:

#### Core

```groovy
implementation "co.zsmb:rainbow-cake-core:0.3.0"
```

The essentials of the architecture library: the view state mechanisms, the dependency injection setup, and configuration entry points.

#### Navigation addon

```groovy
implementation "co.zsmb:rainbow-cake-navigation:0.3.0"
```

All the navigation and argument handling features. `navigator`, `SimpleNavActivity`, and more.

#### Channels addon

```groovy
implementation "co.zsmb:rainbow-cake-channels:0.3.0"
```

Coroutine channel support in the form of - most notably - the `ChannelViewModel` base class and the `LiveData#toChannel` extension.

#### Timber addon

```groovy
implementation "co.zsmb:rainbow-cake-timber:0.3.0"
```

You only need this artifact if you want the framework to log about its internal events (this is mostly just the exceptions caught by `JobViewModel`), and you want it to do so using Timber. For more details, see [Configuration](/features/configuration/).
