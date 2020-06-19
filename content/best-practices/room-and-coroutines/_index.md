+++
title = "Room & coroutines"
weight = 20
+++

Room is a great example of a database library that can be used with coroutines (similarly to [how Retrofit can be used with coroutines](/best-practices/retrofit-and-coroutines)).

Methods on a Room Dao interface can be declared as suspending methods, and are safe to call even from the main thread (though in RainbowCake, you should already be on the IO dispatcher by the time you reach the data layer, see [Threading](/getting-started/tutorial/theory-threading/)). In this case, execution of your database operations will be happen on a background thread managed by the architecture components libraries.

```kotlin
@Dao
interface FlightsDao {
    @Query("SELECT * FROM flights")
    suspend fun getAllFlights(): List<Flight>
}
```

If you want to receive continuous updates from your query as the contents of your database change, you can request a [`Flow`](https://kotlinlang.org/docs/reference/coroutines/flow.html) as the return type of your method, wrapping the original result of your query. This will emit the current state of the database when you first start collecting it, and then emit again if the underlying table changes.

>Note that a method querying for a `Flow` of data doesn't have to be (and shouldn't be) a suspending method. Methods that return a `Flow` will return immediately.  

```kotlin
@Dao
interface FlightsDao {

    @Query("SELECT * FROM flights")
    fun getAllFlights(): Flow<List<Flight>>

}
```
