+++
title = "Flows"
weight = 20
+++

One-time suspending data fetches from data sources using the [`execute`](/features/viewmodel-execute/) method in your ViewModels is a simple way to populate your screens with data.

However, there are cases where this isn't quite enough:
 
- You may receive information from data sources continuously, such as location updates, incoming messages on a websocket connection, and so on.
- The data you've fetched can change in the meantime, and you'd want to get notified about these changes, so that you can keep the UI up-to-date.

A neat way to do this is by using [coroutine Flows](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/).

### Creating a Flow

First of all, you'd have to create a `Flow` somewhere in one of the Data sources of your application, to expose data from it. If you need to do this manually, [`callbackFlow`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/callback-flow.html) will often be the way to go.

However, there are also libraries that already support the `Flow` type, notably, Room. If you return a type wrapped in a `Flow` from your Room queries, it will re-emit the results into that `Flow` when the contents of the database table change.

>The example shown here is from the [GuardianBusinessNews](https://github.com/rainbowcake/guardian-demo) sample app.

```kotlin
@Dao
interface NewsDao {
    @Query("SELECT * FROM newsitems")
    fun getAllNewsItems(): Flow<List<RoomNewsItem>>
}
```

Note that a method querying for a `Flow` of data doesn't have to be (and shouldn't be) a suspending method. Methods that return a `Flow` will return immediately.

From here, you can return the Flow from your Data source, your Interactor, and your Presenter as well.

If you need to perform operations on the Flow of data, you can add those at any layer. For example, you can transform database specific models into domain models in your Data source, and domain models to screen specific presentation models.

```kotlin
class DiskDataSource @Inject constructor(
    private val newsDao: NewsDao
) {
    fun getSavedNews(): Flow<List<News>> {
        return newsDao.getAllNewsItems().map { news -> news.map(RoomNewsItem::toNews) }
    }
}
```

>For more about mapping in RainbowCake, see [Theory: Models](/getting-started/tutorial/theory-models/) and [Mapping code style](/best-practices/mapping-code-style/).

If you do perform operations on the Flow along the way, you'll want to make sure that you're doing it on the correct thread. Since the are no suspending calls here, you don't have to call `withIOContext` in your Presenter. However, that's a good place to specify where the Flow should... flow.
 
This can be done with the [`flowOn`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flow-on.html) method, which will make sure that any operations called on the Flow upstream are executed on a background thread:

```kotlin
class SavedPresenter @Inject constructor(
    private val newsInteractor: NewsInteractor
) {

    fun getSavedNews(): Flow<List<SavedNewsItem>> {
        return newsInteractor.getSavedNews()
            .map { news ->
                news.map(News::toSavedNewsItem)
            }
            .flowOn(Dispatchers.IO)
    }
}
```

A note about testability: in the future, RainbowCake will likely provide a utility method for flowing in the IO context, which will also have support in the `rainbow-cake-test` artifact. For now, if you're using `flowOn` and want to test your Presenters, you'll have to inject a Dispatcher yourself, so that you can replace the one being used during tests.

### Collecting from a Flow

The previous section covered getting a Flow all the way to the ViewModel, let's see how these values can be collected and put to use.

In the ViewModel, you can use [`collect`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/collect.html) to process the values of the Flow, which is a suspending function. This means that you have to call it from inside a coroutine.

The most important thing to remember here is that *`collect` will suspend until the entire Flow is collected*. This means that collecting values inside an `execute` block will prevent any other actions from being processed, until the `Flow` ends (if it ever does).

This means that you should use the non-blocking `executeNonBlocking` method instead. If you want to get updates from the `Flow` for the entire lifetime of a ViewModel, you can do this in an `init` block, like so:

```kotlin
class SavedViewModel @Inject constructor(
    private val savedPresenter: SavedPresenter
) : RainbowCakeViewModel<SavedViewState>(Loading) {

    init {
        executeNonBlocking {
            savedPresenter.getSavedNews().collect { news ->
                viewState = if (news.isEmpty()) {
                    Empty
                } else {
                    SavedReady(news)
                }
            }
        }
    }
}
```

Inside the `collect` call, it's up to you to decide how to react to the values you receive. You can make updates to the `ViewState`, or send events to the View layer. The code inside `collect` is executed on the UI thread.

### Recommended reading

Here's some more recommended reading on Flows:

- [Asynchronous Flow - Kotlin Programming Language](https://kotlinlang.org/docs/reference/coroutines/flow.html)
- [Cold flows, hot channels](https://medium.com/@elizarov/cold-flows-hot-channels-d74769805f9)
- [Simple design of Kotlin Flow](https://medium.com/@elizarov/simple-design-of-kotlin-flow-4725e7398c4c)
- [Kotlin Flows and Coroutines](https://medium.com/@elizarov/kotlin-flows-and-coroutines-256260fb3bdb)
