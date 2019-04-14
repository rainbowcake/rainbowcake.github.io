+++
title = "Models"
weight = 20
+++

There are times when you can get away with taking the data classes returned by your API through your application and all the way to the UI. This, however, does not work well for large, complex applications with multiple data sources and multiple views of the same data. For detailed reasoning, read [this article by Joe Birch](https://overflow.buffer.com/2017/12/21/even-map-though-data-model-mapping-android-apps/), but here's how we'll be handling model objects and mapping between them.

![Models](/images/arch_models.png)

First, each data source may have its own internal models. For example, the disk data source's internal model objects can be marked as a Room `@Entity`, or inherit from `RealmObject`. The network data source can have its models annotated with `@SerializedName` annotations for Gson or with `@Json` annotations from Moshi.

Data sources must not expose these internal models, however. They may only return and receive as parameters domain models (and common, primitive types). These are what interactors perform business logic on.

Finally, each screen gets its own presentation models. These are nested data classes in the presenters. Presenters create these from the domain models returned by the interactors they call.

Read more about [models](/content/concepts/models.md) and [mapping code style](/content/datasources#mapping-code-style).
