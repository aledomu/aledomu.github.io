---
title: "Small progress update of my 2021's GSoC with Fractal-next"
date: 2021-07-02 14:20:00
category: "gnome"
---
This is a small update about the progress of my internship in GNOME for Fractal.
There's not much to tell since I've been mostly learning about the tools
required to complete the tasks.

After some time outlining the design and the path for the implementation of
multi-account support in Fractal-next, I started hacking my way to this goal.
Before I could even start, I had to learn and play around with the GtkBuilder UI
XML format, GLib subclassing in Rust and the whole `ListModel` stuff. As someone
who relies a lot on the type system to get things done and is used to somewhat
linear data flows, this has been quite a bit more difficult than I expected
initially to get a grasp on.

The course of action in the UI will be to have a `GtkPopover`, which will allow
to log in to another account or select an already logged in one. That can be
entirely done through GtkBuilder UI XML files and is already finished. What's
missing right now is the management of the accounts and the list of them behind.
That's what I've been struggling with but it's not far off from finished. For
the `ListModel` required to show the accounts I will take the `SelectionModel`
from the `.pages()` method in `GtkStack` that holds all the `Session` widgets
and pass that as a reference to the `ListView` that actually creates the widgets
to switch accounts, so it's automatically in sync with the changes made to the
stack widget.

After that, I will look for rough edges in the experience and unintended
side-effects that might need some workarounds or modifications to the backend.
But that will happen on the next update post, with screenshots of the results.
