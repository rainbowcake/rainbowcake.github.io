+++
title = "Formatters"
weight = 20
+++

Presenters transform domain models to presentation models that are readily displayable on UI. This might involve assembling text from string resources, which requires access to a `Context`. To keep them easily unit testable, it's best not to inject a `Context` directly into Presenters. Instead, you can introduce Formatters. These are small helper classes that are injected into a Presenter:

```kotlin
class MessagingPresenter @Inject constructor(
        private val messagingFormatter: MessagingFormatter
) {

    suspend fun getStatus(): String = withIOContext {
        val count = messagingInteractor.getMessageCount()
        messagingFormatter.formatStatus(messageCount = count)
    }

}
```

The formatters themselves are free to depend on a `Context` or a `Resources` instance: 

```kotlin
@Singleton
class MessagingFormatter @Inject constructor(
        private val context: Context
) {

    fun formatStatus(messageCount: Int): String {
        return if (messageCount == 0) {
            context.getString(R.string.status_no_messages)
        } else {
            context.getString(R.string.status_you_have_x_messages, messageCount)
        }
    }

}
```
