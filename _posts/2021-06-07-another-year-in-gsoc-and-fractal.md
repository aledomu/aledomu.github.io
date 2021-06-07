---
title: "Another year in GSoC and Fractal(-next)"
date: 2021-06-07
category: "gnome"
---
This year I applied for Google Summer of Code again and chose Fractal to work on
multi-account support. I got accepted (that's why I'm writing this), so today I
start the coding (and design) period to achieve that.

Any of you who had followed what happened in my internship in 2020 and
afterwards might remember that I had the same goal back then. The problem was
that the way the app was structured internally made it incredibly difficult
to do so without all hell breaking loose. You can remind (or read) the details
[in my final report last year](/gnome/fractal-gsoc-final-report/) and what I
learned in the process.

After that, I kept on integrating
[_matrix-rust-sdk_](https://github.com/matrix-org/matrix-rust-sdk)
in Fractal and removing completely the old backend. That was
[completed in January this year](https://gitlab.gnome.org/GNOME/fractal/-/merge_requests/684),
but after fixing all the shortcomings in the code that dealt with the server
(and introducing new bugs related both to Synapse not fully conforming to the
official specification and a few bugs in _matrix-rust-sdk_), I tried to work
further to enable the encryption machinery in the backend, but the undertaking
to do so and make the UI workable at all was much greater than we expected
previously. My #1 concern when I started on this project was maintainability, so
I started thinking that probably the only sane way out was to rewrite Fractal,
something that I suspected since August when I still was in the middle of GSoC 2020.
The fact that GTK 4.0 was just released, its binding crates had better
support for using XML templates and subclassing and we already had proof that
_matrix-rust-sdk_ could work for our needs made it a very compelling
alternative, "just" needing to rewrite the UI from scratch.

This happened more or less when Julian Sparber started working on encryption
support, so whatever the choice was, it had to allow him to focus on his task as
soon as possible. It was deemed that keeping with the incremental refactors
would be much slower for everyone, so Fractal-next got started with proper
support from third-party libraries. Julian started working on it, getting a
basic chat feature set going on in a few weeks that allowed him to get part of
his goal done in parallel. [He posted what he did and an overview of the
architecture of Fractal-next in his blog](https://blogs.gnome.org/jsparber/2021/05/07/the-internals-of-fractal-next/).
I looked around occasionally just to give some advice at the beginning, mostly
concerned with code organization and modules.

One big milestone ahead to make Fractal-next just be _Fractal_ is to reach
feature parity. That's something that Kai will work on as part of his GSoC
internship, as he explains [in his blog](https://blog.kaialexhiller.de/). In the
meantime, I will add support for logging in with multiple accounts. I think we
won't clash each other, so we can do our own thing independently.

Hopefully by autumn we should have a really nice release that brings new
features that the community has been looking for in a long time and make the
project much more future-proof.
