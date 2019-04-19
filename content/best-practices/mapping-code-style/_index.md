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
fun RoomRecipient.toDomainRecipient() = Recipient(
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
