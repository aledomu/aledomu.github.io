---
title: "Refactoring Fractal: Remove Backend (I)"
date: 2020-06-12
category: "gnome"
---
After a week and a half of starting work on Fractal in the GSoC and figuring things out, I could remove all state from one half of the backend, or what is called in Fractal as `Backend`.

Confusing, right? Let me explain further.

Actually the core of the application is split between two structs: one called `AppOp`, where most of the data is managed, and another one called `Backend`, out of the app crate, in _fractal-matrix-api_, where the calls to the server are done. They communicate between through message passing, but `Backend` stores some state that isn't present in `AppOp`, or it's even duplicated. So there are two sources of truth for state.

That makes the process of implementing multi-account support harder and more error-prone than it should be.

There are two paths to the solution here: remove `AppOp` and move all data to `Backend` or do the same in the opposite direction. I chose the latter because I wouldn't have to transfer as much state as in the former case. Moreover, this way I can remove both loops and spawn threads directly and call functions directly from it instead of passing messages and matching against them (while spawning new threads anyways). Beware that these threads are kernel threads, not green threads or coroutines (aka _Futures_), so this is a very grotesque way of doing network requests without blocking the GUI as it is currently. It's something that will be tackled in the future, though.

Put this way, the task sounds simple to conceptualize but really hard to do without mistakes. It turns out it's not that difficult as it would be in a dynamically typed language or any other language without racyness-less guarantees. In Rust, the motto "fearless refactoring" exists for a good reason.

When I started doing the job it was a matter of removing fields in `BackendData` and add them back to `AppOp` (or not, if that was duplicated information) and the fields in the messages used to communicate `AppOp` and `Backend`. The compiler would complain in a helpful manner if I did anything wrong, so I made this part of the task by letting it drive me.

The time came when I had to think what to do with the crude thread pool implementation that was there and a cache for some data isolated from the rest of the app. First things first: take those objects as arguments in the functions that made the requests and have them in `AppOp`. However, this time the procedure was not so obvious. My first attempt, propagating the objects through the functions called from `AppOp`, was a complete disaster. I had to start over and store the objects in the widgets where they were required to make the request functions work. In order to signal in the code where proper async functions (or methods) could be written in the future, I removed them from the widget structs (given that Rust doesn't support inheritance, many widgets in Fractal are built using a delegation-based pattern) and modified the signature of the methods to take the objects as input variables, back to `AppOp` Finally, it worked, but many files were involved and the code got even noisier in general, although the data flow became more explicit.

The merge request with the modifications is [here](https://gitlab.gnome.org/GNOME/fractal/-/merge_requests/581).

The last blocker towards the complete removal of `Backend` is making all the remaining functions return a single value, instead of multiple responses to the app as it does now. In the process, all response (`BKResponse`) variants are going to become only error variants, which could serve as the foundation for a better error system within Fractal, as it's very lacking right now.