+++
title = "Wrap-up"
weight = 60
+++

This is where we'll end the basic example of the architecture for now. The data sources that we didn't look at may be implemented in any way, using any libraries, as long as they have interfaces that only use domain objects as their parameter and return types.

A simple example you can check out for these layers is the (sadly non-public) [NYTimes demo](https://gitlab.autsoft.hu/AutSoft/AndroidChapter/rainbow-cake/rainbow-cake-nytimes), which uses Room for storage and Retrofit/Moshi with a suspending coroutine adapter for network access.

---

You now know the basics of the architecture, now it's time to dive in to the depths of the features that it offers. You can browse the [Features](/features/) section, read up on [Best practices](/best-practices/), and when something really piques your interest, have a look at the [Implementation details](/implementation/).
