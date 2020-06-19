+++
title = "Events"
weight = 20
+++

<div class="small-subtitle">rainbow-cake-core</div>

>For the ideas behind RainbowCake's event handling implementation, see [Thoughts about Event Handling on Android](https://zsmb.co/thoughts-about-event-handling-on-android/).


>This event handling implementation is also covered in [Handling View State and Events with RainbowCake
](https://zsmb.co/talks/#handling-view-state-and-events-with-rainbowcake).

Sometimes lower layers might produce results that don't change the long-term, persistent UI state, but instead they should be propagated to the Fragment just once.

For example, an error might have happened, and you want to produce a one-time toast message about it. you don't want this Toast to reappear every time you put the app in the background and then the foreground again, or every time the screen is rotated - as it would happen if it was simply stored as part of its view state and handled in the `render` method.

Navigation is another very frequent use case for these sort of events - you might do a bit of asynchronous processing based on user input (e.g. perform a save after a button click), and only when it's done proceed to navigate somewhere else. Again, you don't want this navigation to be triggered every time you arrive to this Fragment, so it can't just be a part of its view state.

These events are represented as the `OneShotEvent` type, and can be launched from a ViewModel using the `RainbowCakeViewModel`'s `postEvent` method. If they have no parameter, they can be represented as a simple `object`. Otherwise, simple classes with the parameters declared in their constructor as properties are recommended (there's no real need for a data class' complexity and overhead here).

```kotlin
class DataViewModel @Inject constructor(
        private val dataPresenter: DataPresenter
) : RainbowCakeViewModel<DataViewState>(DataViewState()) {

    object NavigateSuccessEvent : OneShotEvent
    class NavigateFailureEvent(val errorMessage: String) : OneShotEvent

    fun saveData(data: Data) = execute {
        val result = dataPresenter.saveData(data)
        when (result) {
            is Success -> postEvent(NavigateSuccessEvent)
            is Failure -> postEvent(NavigateFailureEvent(result.message))
        }
    }
}
```

Handling these events is done by overriding the `RainbowCakeFragment`'s `onEvent` method, very similarly to `render`. This isn't required to be overridden by default, as not all screens will receive events of this kind.

```kotlin
class DataFragment : RainbowCakeFragment<DataViewState, DataViewModel>() {
    // ...
    
    override fun onEvent(event: OneShotEvent) {
        when(event) {
            is NavigateSuccessEvent -> navigator?.add(SuccessFragment())
            is ShowFailureEvent -> showMessage(event.errorMessage)
        }
    }    
}
```

Any unhandled events in Fragments will be logged, if RainbowCake's logging is enabled (see [Configuration](/features/configuration/)).

### Active observer only events

Events that should only be delivered immediately should implement the `OneShotEvent` marker interface, and be sent using `postEvent`, as described above (note that this method can only be called from the UI thread). If you send one of these events when the `Fragment` is not active, it will never be delivered.

![Active only events](/images/events_active_only_event.png)

*99% of the time, this is the behaviour you need for your events, and the type of events you should use.* 

### Queued events

Events that matter even if they can't be delivered immediately have to implement the `QueuedOneShotEvent` marker, and be sent using `postQueuedEvent`. If the observing `Fragment` isn't currently active, the event will be queued, and all queued events will be delivered immediately when the `Fragment` becomes active again.

![Queued events](/images/events_queued_event.png)


Each `Fragment` instance has its own independent queue of events. Note that `Fragment`s in the background can be destroyed and recreated by the framework, and their queues _will be lost_ in this case - this is a best effort mechanism.

![Events with shared ViewModels](/images/events_separate_queues.png)
