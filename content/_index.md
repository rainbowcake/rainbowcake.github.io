+++
title = "Home"
weight = 0
+++

# RainbowCake - a modern Android architecture

## Introduction and overview

This is the documentation of the RainbowCake architecture concept and implementation, based on Kotlin languages features and the Google Jetpack libraries.

The goals of this architecture as of now:

- Clearly separate concerns between different layers and components
- Always keep views in a safe and consistent state with ViewModels
- In addition to a consistent state, also enable emitting one-time events from these ViewModels (alerts, navigation, error messages) for the views to display
- Handle configuration changes (and even process death) gracefully
- Make offloading work to background threads easy and avoid callbacks
- Provide Fragment-based navigation with argument passing between screens in a single Activity

![Architecture overview](/images/arch_overview.png)

A brief description of the architecture's layers, for a start:

- Views (Fragments or Activities) represent application screens. They observe immutable state from their respective ViewModels and display it on the UI. They also forward input events to the ViewModel, and may receive state updates or one-time events in return.
- ViewModels store the current state of the UI, handle UI related logic, and update the state based on results received from presenters. They start coroutines for every task they have to perform (triggered by input events), and forward calls to their presenters.
- Presenters put work on background threads and use interactors (one or more) to access business logic. Then, they transform the results to screen-specific presentation models for the ViewModels to store as state.
- Interactors contain the core business logic of the application. They aggregate and manipulate data and perform computations. They are not tied to a single screen, but instead group functionality by the major features of the application.
- Data sources provide the interactors with data from various origins - local database and file system, network locations, key-value stores, system APIs, resources, etc. It's their responsibility to keep their stored data in a consistent state (i.e. not expose operations that can lead to inconsistency).

Core technologies used:

- Kotlin & coroutines
- Google architecture components
- Dagger 2

### A grain of salt

This architecture concept and this guide isn't gospel. What's being described here is the full capabilities of the architecture and some design recommendations that are known to work well with it. If your specific application's needs require you to deviate from this, please do. Read more [here](/theory/simplification/).
