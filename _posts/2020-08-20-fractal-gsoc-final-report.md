---
title: "Fractal: GSoC final report"
date: 2020-08-20
category: "gnome"
---
The GSoC is coming to an end and so the work planned for this event for all students. And I'm one of those students.

In previous stages of the development in this event I went full-on to refactor the whole app-backend separation and interaction to decrease the level of indirections when making requests to the server and so have a nicer time adding multi-account support. You can see what I made [here](/gnome/refactoring-fractal-remove-backend-i/) and [here](/gnome/refactoring-fractal-remove-backend-ii/).

As I announced in the [last progress update](/gnome/fractal-july-update-progress/), where I got to rework the error system internally, I started working towards the goal of integrating matrix-rust-sdk into Fractal instead of implementing multi-account support, since the latter came to be a lot more unwieldy than initially thought, having to touch too many moving (and undocumented) parts across the code. But this brought the need to unexpectedly learn another library and get to gripes with its assumptions about its usage.

After a quick glance over the documentation I thought I was ready to tackle the task. First barrier: the library is still early alpha and there was a version conflict with another crate. With some try and failure I got Cargo to accept the setup and I could make the matrix-sdk `Client` login. But initially I had to have two clients working at the same time while I incrementally moved everything to matrix-sdk. Fortunately, there was a method to set the login parameters in the client without making the request to the server. That meant I could share the access token. Neat.

Then I started rewiring the syncing code to use matrix-sdk and take advantage of all its state management machinery for free (with load and save support from disk). Another round of try and failure, this time lots of it, but I could not go forwards by any means. The existing Fractal sync mechanism has many scattered things built in an apparently non-principled manner and no documentation that helped understand the logic behind it (yes, I modified that the last month, but I just replicated what was already working in a way that could work in a single return without a dispatcher to send multiple messages to). Several days went to waste.

I realised I didn't understand well enough how matrix-sdk was supposed to be used. I asked in their Matrix channel and they gave me a few examples. I went through at a slower pace this time and I got the points where I could bypass the state management stuff and get the server responses directly, so I immediatly got to replace Fractal's own API bindings with the ones in matrix-sdk.

The current status of the project is that the matrix-sdk Client is effectively integrated and all the methods for specific requests in `Client` are used except the sync, but it panics given that it needs the Tokio runtime. The WIP merge request is [here](https://gitlab.gnome.org/GNOME/fractal/-/merge_requests/617). What's left in Fractal is the following:

- Re-implement all the remaining requests by using the `.send(request)` method in `Client` and all the API bindings in matrix-sdk instead of the ones in Fractal.
- Get syncing to work with `matrix-sdk` (this will need coordination with other contributors for sure).

There's more work required on the side of matrix-sdk in order for things to fit:

- There needs to be support for pluggable http clients that implement some trait (in the same fashion as it's done for the state storage) in order to make the http client in Fractal work as it does and allow to share it among multiple matrix clients (each one equals to a session).
- Make matrix-sdk runtime-agnostic. If that's not possible, see how Tokio could be made to work in Fractal.

I can't finish without saying the lessons I learned while doing this:

- Always document code from early stages of development, even if it's not a library.
- Don't rush at learning new things, even if they seem easy at first.
- Don't try to make something work too many times, you will waste your time. That's a sign of something that needs to be fixed elsewhere.
- Being ambitious is good, but being able to be realistic at the same time is priceless. But in order to do that you need to know what you are facing, ergo, documentation.
- Working on "infrastructure" code is ungrateful but pays itself on the long run. Technical debt is a thing.
- Writing and releasing an application before all the tooling and library ecosystem around it is ready is a very risky venture that will likely need a deep revision in the future. But that ecosystem needs to be tested by an application to get it to be "completed", so it's a chicken-and-egg problem.
- Try to write as little app-specific code as possible and delegate to libraries with "standard" code (or create your own). That is to say, subdivide the domain problem and partition the code in that way with clear boundaries. The FLOSS community is a unified resource pool of work (with volunteers); allow it to function as such.
- Sometimes the existing libraries are unmaintained, as it was the case with Fractal when it started development: push hard for it, even taking maintainership if necessary (and possible), forking or creating a library from the ground up with similar abstractions. In the worst case, with no existing library, create it in a way that is generic enough so others can use it as well. You might fail at realising how a good abstraction should be: that failure is still valuable knowledge.
- It's not competition that brings real improvements and understanding to the table: **_it's experimentation and taking a reasonable risk to fail_**, which sometimes takes the form of competition. I think this is by far the greatest thing I have learned from this experience.

Some intuitions about Rust that I confirmed:

- Type inference works wonders to avoid most of the usability pitfalls that traditional typed languages have over dynamically typed languages, while forcing that extra cognitive load where it really matters (but it usually gives useful context when reading code, so it's kind of a pro and not a con at all).
- Duck typing will never replace generics. Yes, the latter works, but it doesn't give any hints about the design and mental model behind the code.
- Strongly typed languages always pay off in the long run despite all the boilerplate sometimes, and they are a godsend when doing moderate refactors.
- Affine type systems (aka ownership system) improve locality of data management in the code in a way that makes incremental **and** simultaneously drastic refactors possible, but they greatly restrict the ways you can do GUIs (which is one of the weakest and yet mostly unsolved domain problems of Rust to this date).
- Because of the last two, Rust lends itself to program in a very disciplined dataflow-centric fashion where abstractions don't leak both in space _and time_ while being absolutely low-level if required (DSLs included through macros), allowing it to cover all the levels of abstraction across the stack, ease learning good discipline and a nice mental model to new (and a bit stubborn) developers that translate to good practices in other languages and making life absolutely miserable for those developers that have quite a bit more experience on imperative languages and their traditional idioms. I don't think I could have achieved to refactor Fractal up to this point in a "traditional" language.