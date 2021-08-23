---
title: "Multi-account support in Fractal-next: GSoC final report"
date: 2021-08-23 17:30:00
category: "gnome"
---
After another month of work and getting a bit of a deeper hang of some GTK4
mechanisms like `GtkExpression`, the 2021 edition of Google Summer of Code comes
to an end.

In previous posts I explained my journey towards being able to implement an
account switcher, using the new `ListModel` machinery in the end. While I
already worked on Fractal in 2020, this time I did my task over a clean slate,
given that a complete rewrite of Fractal was started.

The design outlined by Tobias Bernard has been implemented as he said, with the
exception of multi-window support. However, it is not fully functional yet given
the appearance of two weird bugs. The most notable one is that when clicking
over any user entry it does not change the `GtkStack` page even though the
signal handler calls the appropriate method. Initially it worked right, but this
bug got in the code out of nowhere in the middle of the development process.
Another issue is that multiple user entries in the switcher can appear as
selected at the same time. Both problems have been diagnosed for days both by me
and Julian Sparber and we found nothing clear. We are not even sure if we are
hitting a bug in GTK4. I will update this section when we discover what's going
on and fix the issues. Once that is tackled, the main MR will be merged and this
work will be part of Fractal.

My greatest discovery in all this process is the one-way data binding mechanism
that has been introduced in GTK4: `GtkExpression`. It is a very convenient way
of expressing data relationships between application data and the widget tree,
or between widgets on unrelated places of the hierarchy. However, sometimes it
becomes a bit unconfortable due to the lack of error reporting when it does not
behave as expected.

As I said in my previous post, GTK from Rust does not fully exploit the type
system and makes development more error prone, having to rely more on manual
testing of edge cases which are difficult to think of when you are now to a
library/framework.

Notwithstanding these shortcomings, a few auxiliary MRs have been accepted. Here
is the full list:

* <https://gitlab.gnome.org/GNOME/fractal/-/merge_requests/793>
* <https://gitlab.gnome.org/GNOME/fractal/-/merge_requests/794>
* <https://gitlab.gnome.org/GNOME/fractal/-/merge_requests/801>
* <https://gitlab.gnome.org/GNOME/fractal/-/merge_requests/802>
* <https://gitlab.gnome.org/GNOME/fractal/-/merge_requests/774> (the main one)

I cannot thank enough the opportunity and help given by GNOME, Julian, Tobias
and Daniel García, my mentor.
