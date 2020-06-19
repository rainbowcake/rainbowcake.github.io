+++
title = "Retrofit & coroutines"
weight = 20
+++

Retrofit is a good example of a network data source library that can be used with coroutines (similarly to [how Room can be used with coroutines](/best-practices/room-and-coroutines)).
 
The best way to do this is to [directly declare suspending methods in your interfaces](https://zsmb.co/retrofit-meets-coroutines), which is available since `2.6.0`:

```kotlin
interface FlightsApi {
    @GET("flights")
    suspend fun getFlights(): List<Flight>
}
```

- The network call still has to block a thread at some point, but this won't be the UI thread, nor one of our IO threads. The API will suspend the coroutine on the IO thread we called it from and execute network calls on the default OkHttp threadpool (the same one that would've been used if we had a callback-based Retrofit API implementation with `enqueue`).
- These solutions support cancellation as well. If the coroutine is cancelled from the ViewModel, this cancellation will immediately run down the suspending call chain all the way to the data source, and cancel the network call directly and immediately. This means that if we leave a screen while a long network call is ongoing, the call won't still be completed in the background just to have its result thrown away when it's done, like with some other threading solutions.

### Notes

An older, alternative solution was to use a suspending call adapter that returns a `Deferred`, such as [this one](https://gist.github.com/zsmb13/c539cbce5ca9b85d9502436f2f286605) or [this one](https://github.com/JakeWharton/retrofit2-kotlin-coroutines-adapter). There is no need to use such an adapter any more - you should just declare suspending functions directly.
