---
title: "Fractal: Update progress"
date: 2020-07-22
category: "gnome"
---
It's being a busy month, but productive nonetheless!

Since the last update about how things were going most of the error handling stuff has been reworked, as announced. There are a few bits remaining but they are in very specific places that require prior work in other areas. The approach chosen was to have a common trait that handled the error and each backend function now has a (mostly) specific error type that implements that said trait. Managing errors for new requests is as easy as creating a new type for the error that indicates all possible cases, composing over foreign error types if required, and implementing the trait `HandleError` to manage how the error should be shown in the GUI and/or logged, or just marking the trait if the default implementation is good enough. The MR requests that led to this work are:

* [https://gitlab.gnome.org/GNOME/fractal/-/merge_requests/596](https://gitlab.gnome.org/GNOME/fractal/-/merge_requests/596)
* [https://gitlab.gnome.org/GNOME/fractal/-/merge_requests/600](https://gitlab.gnome.org/GNOME/fractal/-/merge_requests/600)
* [https://gitlab.gnome.org/GNOME/fractal/-/merge_requests/603](https://gitlab.gnome.org/GNOME/fractal/-/merge_requests/603)

This allowed me to further cut down the error module, which became too broad to get useful diagnostics.

There has been other changes as well: [replacing the custom `clone!` macro with the one in glib-rs](https://gitlab.gnome.org/GNOME/fractal/-/merge_requests/598) with the ability to avoid manual management of weak references (in the process I spotted a bug in the glib macro related to modules and namespaces, [which I fixed](https://github.com/gtk-rs/glib/pull/662)), [fixing an old bug](https://gitlab.gnome.org/GNOME/fractal/-/merge_requests/599) that messed with the account settings and making the types in the code more strict by [separating URLs from local paths](https://gitlab.gnome.org/GNOME/fractal/-/merge_requests/606) within the same field of some structs and arguments in functions (intermingling with different kinds of data in the same place and type is the perfect recipe for hard to uncover bugs).

There has been some experimentation on the way, like trying to remove `AppOp` as a global static variable and weak reference, but there were many blockers to achieve them. Moreover, as I progressed further, I discovered that the codebase needs too much restructuring to be able to add multi-account support within the summer. I talked with my mentor and we settled on replacing all our custom (de)serialization of the API with the ruma or matrix-rust-sdk library, paving the way for encrypted messages support and implying the removal of fractal-matrix-api. The original plan remains for further work after the GSoC, although [I've opened a WIP merge request](https://gitlab.gnome.org/GNOME/fractal/-/merge_requests/607) for the long term where I'll be sending changes to rework the UI management.