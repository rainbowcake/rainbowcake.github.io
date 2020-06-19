+++
title = "ListAdapter"
weight = 20
+++

The `RecyclerView` AndroidX library includes a [`ListAdapter`](https://developer.android.com/reference/androidx/recyclerview/widget/ListAdapter) helper class that makes it easy to both populate a `RecyclerView` and to make changes to it later, given that you can always pass in the entire list of objects to display.

When given a new list to display, it calculates the difference between the two lists asynchronously on a background thread, and dispatches updates to the `RecyclerView` automatically. This essentially provides the same nice updates that precise `notifyItemXyz` calls would result in, without having to make these calls manually. It also stores the list of items, so there's no need to create a property for the list in the adapter manually.

Here's an example implementation of an adapter that displays the names of a list of users. Notable parts:

- The `ListAdapter` base class takes two type parameters, one is the type of objects it has to store, the other is the usual `ViewHolder` type argument for a `RecyclerView.Adapter`.
- The `UserViewHolder`, `onCreateViewHolder` and `onBindViewHolder` implementations are essentially the same as with a regular `RecyclerView.Adapter`. The only notable difference is that you need to use the adapter's `getItem` method in `onBindViewHolder` to get the list item for the given position.

```kotlin
class UserAdapter : ListAdapter<User, UserAdapter.UserViewHolder>(UserComparator) {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): UserViewHolder {
        val view = LayoutInflater
                .from(parent.context)
                .inflate(R.layout.item_user, parent, false)
        return UserViewHolder(view)
    }

    override fun onBindViewHolder(holder: UserViewHolder, position: Int) {
        val user = getItem(position)
        holder.bind(user)
    }

    class UserViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        private val tvName = itemView.tvName

        fun bind(user: User) {
            tvName.text = user.name
        }
    }

}
```

We've also passed a `UserComparator` object to `ListAdapter`, this is what it uses to calculate the difference between its current list and any new list we pass to it. This argument has to be a `DiffUtil.ItemCallback`, parameterized with the type we store in our list.

It has two methods for us to implement:

- `areItemsTheSame`: whether the two items represent the same object, this is usually decided by comparing the items' unique IDs. They might have different values for other properties, for example, the user's name might have changed, this is of no concern for this method.
- `areContentsTheSame`: whether the two items are exactly in the same in every property. This is most easily computed by using a `data class` for the model and using `==` on it, which makes a call to its generated `equals` method.

```kotlin
object UserComparator : DiffUtil.ItemCallback<User>() {
    override fun areItemsTheSame(oldItem: User, newItem: User): Boolean {
        return oldItem.id == newItem.id
    }
    
    override fun areContentsTheSame(oldItem: User, newItem: User): Boolean {
        return oldItem == newItem
    }
}
```

Finally, we need a way to pass data to the adapter, this is done by calling the `submitList` method:

```kotlin
userAdapter.submitList(viewState.users)
```
