+++
title = "Channels"
weight = 20
+++

<div class="small-subtitle">rainbowcake-channels</div>

The `ChannelViewModel` base class provides the ability to observe updates from channels in a safe and concise way, in addition to providing state handling and coroutine Job execution.

_This API, as the coroutine Channel API itself, should be regarded as experimental._

### Definitions

To discuss the API provided, some terms need to be defined first.

- **Channel:** A coroutine [`Channel`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-channel/index.html) emitting a stream of events. In this use case, they are used to enable the ability of lower layers of the architecture (interactors, data sources) to push events and data to the ViewModel without these events having to be polled for.

- **Observer:** The callbacks that are hooked up to a channel and receive updates (new elements) from it as they're emitted.

- **Observation** An *observation* is a collective name for a channel and the callbacks connected to it via the `observe` methods. There's always a one-to-one relation between *observations* and channels.

### Observation API

The most important provided by `ChannelViewModel` is `observe`, an extension on the `ReceiveChannel<T>` type. The `observe` method **will not return until the observation is over**, so it should almost always be used within a [non-blocking `execute` call](/features/viewmodel-execute/#executenonblocking).

```kotlin
class LocationViewModel @Inject constructor(
        private val locationPresenter: LocationPresenter
) : ChannelViewModel<LocationViewState>(LocationViewState()) {

    init {
         executeNonBlocking {
            val updates = locationPresenter.getLocationUpdates()
            updates.observe { location ->
                viewState = viewState.copy(
                    currentLocation = location
                )
            }
        }
    }    

}
```
#### Important notes

When an *observation* is cancelled, the ViewModel stops receiving updates from its channel **and** the channel itself is cancelled as well.

This means that only a single `observe` call should ever be made to a channel. Subsequently, any classes in lower layers should always return new channel instances when one is requested from them, as they can not be reused due to the way they are tied to the *observations*.

Cancellation also happens the other way around: if a channel is cancelled on the supplier's end, in the lower layer, the *observation* will be cleaned up as well (after running the appropriate callbacks of `observe`).

#### Cancellation callbacks

The trailing lambda seen in the code above receives the elements that the Channel emits. There are two additional callbacks that can be specified to handle the observation ending:

```kotlin
updates.observe(
    onCancelled = { Timber.d("Channel was cancelled") },
    onClosed = { Timber.d("Channel was closed") }
) { location ->
    viewState = viewState.copy(
        currentLocation = location
    )
}
``` 

- `onClosed`: A callback for when the channel is closed. This may be because the data that was being supplied has ran out, because an exception happened somewhere along the way, or because the ViewModel was cleared and therefore cancelled the observation.
- `onCancelled`: Similar to `onClosed`, but will only be called if the channel has terminated exceptionally due to an Exception being thrown in the lower layers. When this happens, `onClosed` will still be invoked, but this callback runs first.

#### Keyed and unkeyed observations

Starting an *observation* in an `init` block is the simplest way to do it. The channel will be requested when the ViewModel is created, observed while it exists, and closed when the ViewModel is cleared. It may also be required to observe a channel more dynamically, starting and stopping the observation based on various events (e.g. user interaction) during the lifetime of a ViewModel.

For these use cases, an optional `key` can be provided to the `observe` method. Keyed observations are unique, two observations with the same `key` can not exist at the same time.

The `replaceExisting` parameter of `observe` can be used to indicate whether to replace the existing *observation* for the given `key` with the one being created now, if it exists. If this parameter is false and the `key` is already taken, the `observe` call is a no-op and terminates silently. The channel that was to be observed will be cancelled.

```kotlin
class LocationViewModel @Inject constructor(
        private val locationPresenter: LocationPresenter
) : ChannelViewModel<LocationViewState>(LocationViewState()) {

    fun startLocationUpdates() = executeNonBlocking {
        val updates = locationPresenter.getLocationUpdates()
        updates.observe(
            key = "location_key", 
            replaceExisting = true
        ) { location ->
            viewState = viewState.copy(
                currentLocation = location
            )
        }
    }

}
```

Note that this is wasteful because getting hold of the channel might have already activated callbacks and other mechanisms in underlying data sources, which will have to be shut down. 

If this activation and shutdown has significant costs in a specific use case, the `isObserving` should be used to check if there's an active *observation* for the given `key` first. A `removeObserver` method is also available to cancel an ongoing observation with a given `key`.

```kotlin
fun startLocationUpdates() = executeNonBlocking {
    if (isObserving("location_key")) {
        return@executeNonBlocking
    }
    val updates = locationPresenter.getLocationUpdates()
    updates.observe( ... )
}

// This method doesn't need to start a coroutine,
// as removing an observer is synchronous
fun stopLocationUpdates() {
    removeObserver("location_key")
}
```

### Producing channels

The channels that are observed in the ViewModel have to be created and supplied with data in the data layer. Depending on the API that provides it and the use case at hand, this channel creation will fall into one of two categories.

#### Single engine sources

A "single-engine" source will be activated just once, when the first channel is requested from it. For any additional requests, it only needs to create a channel. Whatever callback API it activated or data source it connected to will supply all created channels on its own. The source will be deactivated if it no longer has channels to supply with data. 

An example of this could be a GPS data source, which will only activate the Fused Location API once, and when it receives an update from it, send it to all created channels.

To manage the collection of active channels, a `ChannelCollection` interface and a default implementation of it (`SynchronizedChannelCollection`) are provided. While extending this default implementation might be an issue for callback-based APIs where a callback already needs to extend a given callback base class, it can be easily delegated to instead:

```kotlin
class MultiChannelLocationCallback(
        channelCollection: ChannelCollection<Location> = SynchronizedChannelCollection()
) : LocationCallback(), ChannelCollection<Location> by channelCollection {

    override fun onLocationResult(locationResult: LocationResult?) {
        locationResult ?: return
        forEachChannel { it.offer(locationResult.lastLocation) }
    }
    
    // ...
}
```

All channels can be supplied with the data from the callback using the `forEachChannel` method.

The data source implementation can use this callback the following way:

- Whenever a channel of location events is requested, it's created with the `produceInIOContext` function, a simple wrapper around [`produce`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/produce.html) that just sets the context to be used.
- An intermediary channel (`resultChannel`) is created, and added to the callback collection.
- If this was the first channel added, the engine is activated by invoking the Fused Location API. 
- A loop in `produceInIOContext` reads the updates from `resultChannel`, and transfers them to its own channel, now in the appropriate coroutine context.
- Finally, when the observation is cancelled, a check is made to see if there are any channels remaining in the callback. If not, the callback is unregistered from the API. 

```kotlin
class SingleEngineGpsDataSource @Inject constructor(private val context: Context) {

    private val callback = MultiChannelLocationCallback()

    fun getLocationEvents(): ReceiveChannel<Location> = produceInIOContext {
        val resultChannel = Channel<Location>(CONFLATED)
        val request = LocationRequest().apply { ... }

        val firstChannelAdded = callback.addChannel(resultChannel)
        if (firstChannelAdded) {
            startLocationUpdates(request, callback)
        }

        try {
            for (location in resultChannel) {
                send(location)
            }
        } finally {
            val anyChannelsRemaining = callback.removeChannel(resultChannel)
            if (!anyChannelsRemaining) {
                stopLocationUpdates(callback)
            }
        }
    }

    private fun startLocationUpdates(
            request: LocationRequest,
            callback: LocationCallback
    ) {
        LocationServices.getFusedLocationProviderClient(context)
            .requestLocationUpdates(request, callback, Looper.getMainLooper())
    }

    private fun stopLocationUpdates(
            callback: LocationCallback
    ) {
        LocationServices.getFusedLocationProviderClient(context)
            .removeLocationUpdates(callback)
    }
}
```

#### Multi engine sources

The second type is a "multi-engine" source, which sounds more complicated, but is actually simpler to implement. Every time a channel is requested from this source, it will start fetching data and supplying the channel. When the channel is cancelled, the mechanism supplying it is also shut down.

For this example, we'll use the Bluetooth API to scan for devices with various UUID filters. This will require the registration of a new callback for each call to our data source. Since our callback will only serve a single channel, it will receive just this channel as its parameter:

```kotlin
class ChannelScanCallback(private val channel: SendChannel<BluetoothDevice>) 
    : ScanCallback() {
    
    override fun onScanFailed(errorCode: Int) {
        channel.close()
    }

    override fun onScanResult(callbackType: Int, result: ScanResult) {
        channel.offer(result.device)
    }
}
```

The data source implementation is very similar to before, but without the code that was required to manage the channel collection.

- The channel returned from `scan` is created by the `produceInIOContext` function.
- An intermediary channel (`resultChannel`) is created to be passed into the callback, which is created for each call to `scan`, instead of at the class level.
- The Bluetooth API is invoked. 
- `resultChannel` is read from in a loop, the data in it is forwarded into the returned channel.
- When the observation is cancelled, the callback is unregistered from the API. 

```kotlin
class MultiEngineBluetoothDataSource @Inject constructor(
        private val scanner: BluetoothLeScanner
) {

    suspend fun scan(uuid: UUID): ReceiveChannel<BluetoothDevice> 
            = produceInIOContext(capacity = 100) {

        val resultChannel = Channel<BluetoothDevice>(capacity = 100)
        val scanCallback = ChannelScanCallback(resultChannel)

        val settings = ScanSettings.Builder()
                .setScanMode(ScanSettings.SCAN_MODE_LOW_LATENCY)
                .build()
        val filter = ScanFilter.Builder()
                .setServiceUuid(ParcelUuid(uuid))
                .build()

        scanner.startScan(listOf(filter), settings, scanCallback)

        try {
            for (device in resultChannel) {
                send(device)
            }
        } finally {
            scanner.stopScan(scanCallback)
        }
    }

}
```
