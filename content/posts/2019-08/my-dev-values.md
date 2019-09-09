
---
title: "My Development Values"
date: 2019-08-29T22:00:00-07:00
draft: true
---

This post is mostly for me, as a way to organize my thoughts. It's also an easy way for me to let people know what to expect
when you're working with me.  So, here are the things I value in software development.

# Development Process

![The IT Crowd - Roy: "I know it looks like a bit of a mess. But it's actually a very delicate ecosystem."](https://media1.tenor.com/images/0a600caca354ccf52d71cdb015c612c6/tenor.gif?itemid=4983187)

While I don't always follow these steps strictly, there are generally what I strive to do. The main exception is when I'm doing exploratory work -
times when I'm noodling on an idea, just to see if it works, with the explicit intention of never using the code I write in a real application
without at least doing some serious refactoring (I most definitely have a personal repo called "will it compile?" for things like that).

## Documentation first

Not exhaustive documentation, but before I write code, I want to have written down the basics of how to consume what I'm going to write.
If it's a UI, I'll write a very, very high-level guide to the feature. If it's an API, whether in a programming language or an interchange
protocol like REST or GraphQL, I write a small guide on how to consume the feature I'm building.

The important thing here isn't to be comprehensive, complete, or accurate, but to lay out and clarify the main ideas of the feature, key
relationships between different concepts, and important constraints and workflows.

For example, when building the first version of one of my pet projects, [Kotlin-UAPI](https://github.com/byu-oit/kotlin-uapi) 
(a developer-facing tool for building REST APIs at BYU), after sketching out the basic idea in code, I actually wrote a series of tutorials before writing
any code. As I added features, I would write the chapter in the tutorial about that feature, including the actual code that used the feature. Then,
I started actually building the code.  This allowed me to have a clear idea of what I was building and why, and throw away bad ideas early - if I couldn't
explain the idea clearly and concisely, it got thrown out.

## Tests Second

Or, at least, the skeleton of the tests.  I may not write every bit of every test case up front, but I like to at least define the main things I want to
test fairly early.  For example, if I know I have a class called FooBar, I'll build the skeleton of the FooBarTest early, including empty test cases
for all of the main behaviors I expect to have.

I'll end up with something like (Kotlin + JUnit 5):

```kotlin
internal class FooBarTest {

    // nested suite per method/major behavior
    @Nested
    inner class `doStuff()` {
        @Test
        @Disabled("wip")
        fun `does stuff`() {
            // empty until I start the main TDD pass
        }

        @Test
        @Disabled("wip")
        fun `throws if something is wrong`() {

        }
    }
}
```

(Notice the nested suites and descriptive test names)

## Code and tests third

After I've got an idea of what tests I'll want to have, I'll launch into full TDD mode - tight feedback loops, never going more than a minute or two (usually much less) without adding to and re-running my tests.  Red -> Green -> Refactor is my jam.  Call me crazy, but I really do enjoy writing unit tests!

More on testing later.

# Make the right thing the easy thing...

![Guardians of the Galaxy - Rocket Raccoon (hefting a large gun): "I live for the simple things"](https://66.media.tumblr.com/1ad790b854b992f371d9cd7ca0cae419/tumblr_ojudsqpWho1v6wkmgo2_500.gif)

I like to make the correct way to do a thing also be the easiest way to do it. One way to think of this is 'Standardization through Seduction' 
(I wish I could remember who I stole that term from). I want to make the "right" way (in terms of "right for the organization as a whole")
so easy that you'd be foolish not to do it that way.

As an example, if we want developers to be doing CI/CD, we better have a good value proposition for it - how can it make the developer's life 
tangibly easier right here and now? We also better have good tooling around it, and make the getting-started-from-nothing process very easy.

Another example is brand compliance. In my work with the [BYU Theme Components](https://github.com/byuweb/byu-theme-components), I strove to
make sure that it was so easy to make your site be compliant with our web brand that it was more work (usually much more work!) to not be 
compliant. Those who decided to throw the brand guidelines away and do their own thing had a much harder time of it, while those who followed
them could be up and running literally in minutes (Minimum of 3 lines of HTML in the `<head>`, and a few in the body - actual number varied
depending on how many bells and whistles you needed, but probably no more than 30). All this with accessibility, responsiveness, progressive
enhancement, and performance best-practices baked in without any effort on your part.

# ... but have an escape hatch for the weird stuff

![The IT Crowd - Moss: "I want to go back to being weird. I like being weird. Weird is all I've got. That and my sweet style."](https://media.giphy.com/media/q4TJ4EotHZylW/giphy.gif)

Making the "right" way easy is great, but what about the 1% of cases where it's not the right way?  We should make those possible without
starting from scratch. In the [Kotlin UAPI](https://github.com/byu-oit/kotlin-uapi) project, while we took care of most of the HTTP/JSON
stuff for you, there were all sorts of hooks (I never quite documented all of them!) that you could use to customize things.
Don't like how we were parsing your JSON input body? Great! Just override the parent interface's default method that gets a JSON reader, and do 
whatever you want to do! Need to add a custom HTTP endpoint? Easy! We give you a way to get a reference to the underlying HTTP server
so you can add your own routes!

As a counter example, you can look at BYU's [Handel](https://github.com/byu-oit/handel) cloud IaC tool. If Handel didn't explicitly support
something you needed to do, too bad! I kind of managed to alleviate that by adding ["extensions"](https://github.com/byu-oit/handel/issues/302) 
to Handel, but they never really caught on (apparently, being a Node developer and an AWS-certified Solutions Architect wasn't enough for 
people to feel comfortable writing Node code to configure AWS).

# Simple interfaces hiding complex ideas

![The Wizard of Oz - Pay no attention to the man behind the curtain!](https://media1.tenor.com/images/10b05368823742f0fe1c5539226c01ca/tenor.gif?itemid=9488490)

Once again, I point to [Kotlin UAPI](https://github.com/byu-oit/kotlin-uapi). It's a Kotlin 
implementation of BYU's University API
spec, which is a very, very, very complex specification for REST API development. It's pretty far
from simple or easy - in the years since it was launched, I haven't seen anybody write an API that
was actually 100% compliant with the spec - including the trial balloon APIs that were written to
try out the spec!

So, I decided to write the spec compliance part once and never do it again. The point of kotlin-uapi
was to make it so that the API developer didn't have to know much about the spec - they just needed
to implement some interfaces, provide a little wiring, and bam! They've got a UAPI-style API! On our
first production project that used it, thanks to this and several other tools we had developed (like 
[Handel](https://github.com/byu-oit/handel)), we went from a brand-new, empty Git repo to a 
stubbed-out-but-UAPI-compatible API deployed and running in AWS, complete with continuous deployment
and automated health monitoring, in 30 minutes.

I love the challenge of hiding complex things behind a simple interface. In that vein, Kotlin DSLs
are one of my favorite things right now. Also, [Svelte](https://svelte.dev) is awesome.

# Single-purpose tools over invasive frameworks

![MASH - Maj. Winchester: "I do one thing at a time. I do it very well, and then I move on."](https://miro.medium.com/max/900/1*AmzpRNoeigxvkejkIcLtgA.jpeg)

Now, I'm not saying that a framework like Spring is bad, but what I am saying is that 90% of projects
written with Spring are. And you certainly shouldn't have a small utility require Spring in order to
function! As a counterexample to what I like, we once had a Java library for loading encrypted credentials 
from the filesystem and decrypting them using a server-specific secret key. None of what it did
required Spring, but there was no way to use it without dragging the entire Spring framework along!

To paraphrase Maj. Winchester from M*A*S*H - a good tool "does one thing, it does it very well, and then
it moves on."

However, a tool should be complete and non-trivial. left-pad and its ilk are an abomination.

# Security, Performance, and Accessibility from the beginning

I like to start every project with clear security, a11y, and performance goals. Our product can have the best
features, but if it can't work for any user securely and efficiently, it's not going to succeed in the long run.
Either we'll expose sensitive data because we didn't care about security, or we'll give users a bad,
slow, bloated experience that they won't want to use. These three things are everyone's job.

This means we have to make building apps that are secure, accessible, and efficient easier. If you've got an hour or three,
I can fill you in on my ideas for this. It mostly boils down to "write less code" and
"notice issues early".

Frameworks like Svelte and tools like Kotlin coroutines are my performance friends.

# Developers aren't interchangeable



Developers are people too!

I hate the idea that you can hand any developer on a team a task from any part of an application and
expect it to be done well. True full-stack, polyglot developers are few and far between, and you
can't force someone to become one. You'll just burn them out. They've got to become one through lots
of experience, plus some luck and something ineffable that I don't know how to quantify.

This doesn't mean devs shouldn't stretch themselves and take on tasks that are out of their comfort zone.
What I don't like, though, is when a product manager comes, finds the one dev who has the lightest workload,
and dumps a new feature on them without regard for what skills it needs.


