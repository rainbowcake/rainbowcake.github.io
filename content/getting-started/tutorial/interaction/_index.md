+++
title = "Interaction"
weight = 50
+++

Now that we know about mapping, we can wrap up our Presenter implementation, and then move on to the next layer, the Interactor. Our presenter will receive a `UserInteractor` via constructor injection, and use it to fetch data:

```kotlin
class UserPresenter @Inject constructor(
    private val userInteractor: UserInteractor
) {
    suspend fun getUser(): User = withIOContext { 
        val domainUser = userInteractor.getUser()
        return User(
            name = domainUser.name,
            profileImage = user.profileImage?.let { Uri.parse(it) }
        )
    }
    
    // Presentation model
    class User(name: String, profileImage: Uri?)
}
```

The call to our Interactor returns a domain model, which is what we perform business logic on. This may look something like this:

```kotlin
data class DomainUser(
    val id: UUID?,
    val name: String,
    val profileImage: String?,
    val friendCount: Int,
    val balance: Double
)
```

We map this to our presentation model so that we only provide the required data to our ViewModel, and in the appropriate format.

The interactor will use data sources to get us our user, for the sake of the example, let's have a network source, and a local disk data source used for caching.

```kotlin
@Singleton
class UserInteractor @Inject constructor(
    private val networkDataSource: NetworkDataSource,
    private val diskDataSource: DiskDataSource
) {
    suspend fun getUser(): DomainUser {
        val networkUser: DomainUser? = networkDataSource.getUser()
        if (networkUser != null) {
            diskDataSource.saveUser(networkUser)
            return networkUser
        }
        
        return diskDataSource.getUser()
    }
}
```

You can see that the interactor performs pure business logic, and contains no mapping, thanks to the data sources already returning and accepting the same domain model.

Also, since our Interactor method is suspending, any of the data source methods may be suspending as well - but they might also be implemented with blocking APIs.
