+++
title = "Wrap-up"
weight = 60
+++

This is where we'll end the basic example of the architecture for now. The data sources that we didn't look at may be implemented in any way, using any libraries, as long as they have interfaces that only use domain objects as their parameter and return types.

Preferrably, they should also be able to provide suspending interfaces. See [Retrofit & coroutines](/best-practices/retrofit-and-coroutines) and [Room & coroutines](/best-practices/room-and-coroutines) for some examples of how this can be implemented.

A simple example you can check out for these layers is the [Guardian News Sample](https://github.com/rainbowcake/guardian-demo), which uses Room for storage and Retrofit/Moshi for network access.

---

Now that you know the basics of the architecture, it's time to dive in to the depths of the features that it offers. You can browse the [Features](/features/) section, read up on [Best practices](/best-practices/), and when something really piques your interest, have a look at the [Implementation details](/implementation/).
