+++
title = "Mapping code style"
weight = 20
+++

For collections of elements, using the standard library [`map`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/map.html) function is recommended, with explicit arguments, all on new lines. The lambda argument may be named, but the implicit `it` name is also acceptable in these trivial situations.

```kotlin
fun getRecipients(): List<Recipient> {
    return recipientDao.getAll().map {
        Recipient(
                id = it.id,
                name = it.name,
                zipCode = it.zipCode,
                city = it.city,
                street = it.street
        )
    }
}
```

For single elements, using [`let`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/let.html) is recommended, in the same style:

```kotlin
fun getRecipientById(recipientId: Long): Recipient {
    return recipientDao.getById(recipientId).let {
        Recipient(
                id = it.id,
                name = it.name,
                zipCode = it.zipCode,
                city = it.city,
                street = it.street
        )
    }
}
```

This might seem redundant, but it matches the code style of other functions that return nullable, and it also makes it easy to change this function to allow nullable results later on, if necessary:

```kotlin
fun getRecipientById(recipientId: Long): Recipient? {
    return recipientDao.getById(recipientId)?.let {
        Recipient(
                id = it.id,
                name = it.name,
                zipCode = it.zipCode,
                city = it.city,
                street = it.street
        )
    }
}
```

### Mapping extensions

Especially when encountering duplicated mapping code, like shown above with a "get all" and a "get by ID" function that return the same type, it should be extracted to a separate file, in the form of extension functions. For the above example, the following would be placed in a `Mapping.kt` file:

```kotlin
fun RoomRecipient.toDomainRecipient =  Recipient(
         id = it.id,
         name = it.name,
         zipCode = it.zipCode,
         city = it.city,
         street = it.street
)
 ```
 
Using this would simplify the previous functions as such:
 
 ```kotlin
 fun getRecipients(): List<Recipient> {
     return recipientDao.getAll().map(RoomRecipient::toDomainRecipient)
 }
 
 fun getRecipientById(recipientId: Long): Recipient? {
     return recipientDao.getById(recipientId)?.let(RoomRecipient::toDomainRecipient)
 }
 ```


# Retrofit and coroutines

Retrofit is a good example of a data source library that can be used with coroutines. You can either [directly declare suspending methods in your interfaces](https://zsmb.co/retrofit-meets-coroutines) since `2.5.1-SNAPSHOT`, or use a suspending call adapter, such as [this one](https://gist.github.com/zsmb13/c539cbce5ca9b85d9502436f2f286605) or [this one](https://github.com/JakeWharton/retrofit2-kotlin-coroutines-adapter). All of these approaches let us make our calls in a suspending way.

- The network call still has to block a thread at some point, but this won't be the UI thread, nor one of our IO threads. The API will suspend the coroutine on the IO thread we called it from and execute network calls on the default OkHttp threadpool (the same one that would've been used if we had a callback-based Retrofit API implementation with `enqueue`).
- These solutions support cancellation as well. If the coroutine is cancelled from the ViewModel, this cancellation will immediately run down the suspending call chain all the way to the data source, and cancel the network call directly and immediately. This means that if we leave a screen while a long network call is ongoing, the call won't still be completed in the background just to have its result thrown away when it's done, like with some other threading solutions.
