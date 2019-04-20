+++
title = "Theory: Models"
weight = 40
+++

Now, here are some very controversial design choices. The architecture has _a lot_ of mapping between different model objects. Let's look at this visually first:

![Models across the layers](/images/arch_models.png) 

### Domain models

Let's start from the middle. Our business logic is performed by Interactors, and they'll perform this using domain models. These are pure Kotlin data classes for the most part, preferably with all immutable properties. Going both upward and downwards from this layer, we'll be mapping to other models, and here's why.

### Data models

Data sources get their own model objects to use internally. Their interfaces towards Interactors don't expose these models, only domain models and primitives, but they'll map to and from their own data models internally. These might be similar to our domain models a lot of the time, but they might occasionally have the data in a different format, or for example, be annotated with various library specific annotations (`@Entity`, `@SerializedName`, `@Json`), or inherit from library specific base classes (`RealmObject`). Having separate models here ensures that our business logic doesn't directly depend on these libraries.

### Presentation models

In the other direction, we have Presenters. These map the domain models to screen specific presentation models. Why are these necessary then? Again, this isn't just mapping for the sake of mapping. Domain models might be in different formats than what we need to show on screen, and there may not even be a 1-to-1 mapping between our domain models and items that show up on the UI. This additional mapping is also an opportunity to, for example, perform localization on data before presenting it.

Presentation models should be in a format that is readily displayable on the UI, without any more formatting or data processing required. Essentially, we want all of our UI manipulation code to be as simple as this:

```kotlin
authorNameText.text = viewState.author.name
```

Finally, why do we do this in the Presenter? For one, they're screen specific, and we definitely need different presentation models per screen, as there's little chance that two screens display the exact same set of data in the exact same format. Plus, as we've seen before, all Presenter methods run on the IO threadpool, so this mapping isn't blocking the UI thread (like it would if we did this in ViewModels).

### References

For more why mapping is important, read [this excellent article by Joe Birch](https://overflow.buffer.com/2017/12/21/even-map-though-data-model-mapping-android-apps/).

You may also want to take a look at the [mapping code style](/best-practices/mapping-code-style/) section.
