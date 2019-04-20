+++
title = "Simplification opportunities"
weight = 20
+++

Some aspects of the architecture will only be useful in large, complex application. In smaller, simpler projects they might become boring boilerplate that you can do without. You are, in fact, encouraged to cut a few corners here and there, and take just the parts of the architecture that you need.

Here are some ways you might be able to simplify your application. 

### Mapping less

The architecture prescribes [a lot of mapping](/getting-started/tutorial/theory-models/) by default, between at least three layers of different model objects. While this is for good reason, as explained in that theory section, it's also a lot of code to write if your models in the different layers are very similar to each other.

There might also be performance reasons for wishing to reduce mapping, but this tends to be quite rare. Mapping happens on a background thread anyway.

There are two recommended ways of reducing the mapping work necessary:

1. **Using domain models as presentation models.**

    You may propagate your domain models to the UI if they're of the appropriate structure, and you find them easy enough to display as they are. These models should be pure Kotlin classes, so this doesn't leak any library details to the presentation layer. All you're losing is the screen-specific formatting of these models.

    If you find that there are some screens where you do need to reformat or restructure your models for presentation, you can always introduce presentation models for just that specific screen, lazily.

    ![Using domain models as presentation models](/images/simplification_domain_pres.png)

2. **Using data source models as domain models.** 

    This should usually be the disk data source's model, as the database tends to be source of truth for an application, and the database model's structure is less likely to change than that of the network models.
    
    ![Using data source models as domain models.](/images/simplification_db_domain.png)

### Skipping a layer

TODO
