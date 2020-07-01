---
title: "Refactoring Fractal: Remove Backend (II)"
date: 2020-07-01
category: "gnome"
---
So the time came for removing the `Backend` struct finally! The bits that were left in the previous patch have been removed, which were not just state but a `ThreadPool` and a cache for some info. Those were fitted in `AppOp` without too much thought on consistency of it.

But what does this actually mean for the internal structure of the code?

The result is that any state or utility that was needed for requests and modifying the UI is held only from a single place in the app. With it, the loop in `Backend` has been removed as well, and instead of sending messages to the receiver loop from the backend, those are sent from a spawned thread (to keep the UI thread unlocked) that sends the HTTP request directly and retrieves the response. Put in a simpler way, I replaced message passing to the backend loop with spawning threads, which was done anyways in the loop to be able to have multiple requests at the same time.

I acknowledge that doing this kind parallelism with system threads in 2020 is a very crude way of doing the task, to say the least, but using coroutines requires a significant amount of work in other areas of the app right now.

But I didn't stop there. By having replaced message passing to a loop with calling the right function in a separate thread, I could take the other half of the task done in the receiver loop and bring it to the same thread after completion of the request/response function. In the process of doing this, many of the variants of `BKResponse` (the enum that the backend loop sent back to the app loop), leaving only those that carry error information. I didn't do the same for error conditions because many of them were managed in a generic way, and would have duplicated too much code.

That would seem enough, except for the fact that having a loop to manage a single value within the same crate does the same as having a function that processes the data in the same way, except with greater overhead. So I went on and changed the function that setup and ran that remaining loop and repurposed it to act as a dispatcher of errors on a call basis by extracting the giant match that was inside the loop doing the actual work. After that, that dispatcher is called from the same thread where requests are made, without any loop nor message passing involved.

So now not only all state is in AppOp and there is no backend loop, there are no big busy loops at all in the app code and error management becomes simpler. Another win is that less data has to cross thread contexts, with the improvement in performance that it brings (or at least it should, I didn't benchmark it and I don't know the impact it actually has).

The merge request with the modifications is [here](https://gitlab.gnome.org/GNOME/fractal/-/merge_requests/590) and has been already accepted.

The next thing I will do is to tackle that error dispatcher function. I find very problematic to have a single match as big as that one for future extensibility of error management. There is a bit more I have to think but I'm mostly settled on turning all the variants into separate structs that implement a trait that replaces the dispatcher. The functions that do all the request-response stuff would return those structs directly in case of error, which most of them will be dedicated to each function. This would allow as well to completely get rid of intermediate conversions to a common error type for Fractal as it's done now and lose a lot less information.