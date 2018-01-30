---
title: "Bringing Order to Web Design Chaos"
description: "Or, how I learned to stop worrying and love Web Components"
canonical_url: https://www.thatjoemoore.com/posts/2018-01/byu-theme-components/
date: 2018-01-29T21:27:42-07:00
published: false
tags: [web components, design]
---

This is the story of an interesting engineering effort I've participated in over the last year.
It's been a really fun side project for me (~20 hours a week at the start, ~5 hours a week
since then), which has allowed me to experiment with some fun new technologies and
implement some cool stuff. It centers on an effort to bring a simple, unified theme to all of
our many, many web properties that can share code between any CMS or frontend framework.

# Where We Started

I work for [Brigham Young University](https://www.byu.edu), a large, private University.

Universities are a unique environment. In a university like ours, you have a number of
large colleges and schools, all of which have a lot of freedom to do what they want.
Historically, one way in which this freedom has manifested itself is in Web design, which
has lead to some very...inconsistent designs.

For example, this time last year, a designer I work with looked at the home pages of all
of our top-level colleges and schools, and found that they all had completely different
designs. They used different logos, layouts, shades of our school colors (navy blue and white),
and, in one extreme case, didn't use our school colors at all. Many of these pages were loosely
based on a design that had been published in 2013. This particular design was never well documented 
and the reference implementation wasn't portable across platforms, so we saw a lot of divergence
and changes as time went on.

I don't mean to shame any particular schools, because most of their designs actually looked quite
nice, but here's a ~~horrifying~~ fun picture to show what I'm talking about:

{{< figure src="byu-sites.png" title="See anything wrong here?" >}}

After hiring an outside firm to conduct an audit of our web presence, it was decided that
it was time to try and bring some brand uniformity to the University. An initiative was
launched in our web community to design and implement a simple, modern, unobtrusive standard
theme and to encourage developers and designers across campus to converge on this theme.

I'm not going to talk about the design part, because I'm not a designer. While I was asked
to attend the design meetings, it was as an engineer who would be asked to develop a reference
implementation of the theme. I should note, however, that it was decided to only require a
small header (including search and user sign-in), simple navigation, and a footer. Everything
else was up to individual sites to design, though we've also been working on providing them
with off-the-shelf styles and components so that every site doesn't have to redesign common 
elements.

# Our Goals

Once our design group hammered out a design that everyone could work with, it was time for
our engineering group to implement things. While the design group worked, we had one department
that stepped up in a big way and developed a Drupal module that allowed them to rapidly 
iterate on the design across several of their sites, so the designers could get feedback on 
what worked in the real world and what didn't.  However, for long-term use, we decided that
we wanted a solution that gave us three things:

1. Easy integration with any CMS, especially Drupal and Wordpress, as well as hand-written pages and applications.
2. Simple use with very little ceremony and structure.
3. The ability to maintain a single codebase and push updates and bugfixes out to everyone quickly and easily.

A few of us in the group had been playing with Web Components, and the v1 version of the
specifications had just been implemented in Chrome. After looking at several proposals on
how to reach our goals, we decided that Web Components would give us a lot of what we were
looking for.

# Web Components to the Rescue

Web Components solved a lot of our problems. Because they're a platform-level primitive,
we didn't have to worry as much about integration - they can (theoretically) integrate
with anything that can work with the DOM.

To put this in perspective, it took a couple
of student programmers who hadn't seen our code before about 30 minutes to implement
a Wordpress theme that was based on our Web Components. My favorite demonstration
of the ease with which we could integrate came when I was doing a presentation about
the theme to the campus organization I work for. As a live demo, I took one of our really 
old, nasty pages and retrofitted it to use the new theme, right there in front of everyone.
It took about 10 minutes. Granted, I had practiced this beforehand, but it was still
pretty fun to show just how easy it could be.

Beyond the ability to write them out to the DOM, Shadow DOM gave us some really nice
style isolation, so we could be certain that none of the site's styles could accidentally
mess up our themed components, and none of our internal styles would leak out to the
outside world.

There have been some hiccups on the way, but our CMSes have had a pretty easy integration
story, and we've got Single-Page Apps in production that are using React (16+), 
Angular (2+), VueJS, and others. 
There are a handful of SaaS tools doing some truly awful things to the DOM that we haven't
gotten working yet, but for the most part, it's been pretty uneventful.

Web Components also promised to solve some of our complexity problems. If you're familiar
with Bootstrap, you know how, in order to use their styles, you have to structure your markup
in just the right way, with lots of nested `div`s and special classes. It's really easy to get
lost in all of that structure, especially since the design we were given had some very large
layout differences between mobile and desktop views. The concept of slots in Shadow DOM gave
us a really easy way to hide the markup complexity from the consumer, while still allowing
them to plug in all of their site-specific things.

Our final header markup looks something like this:

```html
<byu-header>
    <a slot="site-title" href="some-site.byu.edu">Site Title</a>
    <byu-menu slot="nav">
        <a href="#link1">Link 1</a>
        <a href="#link2">Link 2</a>
        <a href="#link3">Link 3</a>
        <a href="#link4">Link 4</a>
    </byu-menu>
    <byu-search slot="search" action="submit-form">
        <form action="search.php">
            <input type="search" name="search" />
        </form>
    </byu-search>
    <byu-user-info slot="user">
        <a slot="login" href="#login">Sign In</a>
        <a slot="logout" href="#logout">Sign Out</a>
        <span slot="user-name">Joe Student</span>
    </byu-user-info>
</byu-header>
```

At first glance, there's a lot going on here. However, most of the structural complexity
is hidden, and only the things that are specific to a given site are included.

To give you an idea of how much structure has been left out, here's what this looks like after
being rendered by the Shady DOM polyfill, which combines all of the content into one DOM tree:

```html
<byu-header mobile-max-width="1023px" max-width="1200px" class="byu-component-rendered" left-align="">
    <div id="header" class="byu-header-root">
        <div class="byu-header-content needs-width-setting stretches">
            <div class="byu-header-primary">
                <a class="byu-logo" id="home-url" name="home-url" href="https://byu.edu/">
                    <img class="byu-logo" alt="BYU" src="https://cdn.byu.edu/shared-icons/latest/logos/monogram-white.svg">
                </a>
                <div class="byu-header-title">
                    <a slot="site-title" href="some-site.byu.edu" class="">Site Title</a>
                </div>
            </div>
            <div id="secondary" class="byu-header-secondary">
                <div class="byu-header-user">
                    <byu-user-info slot="user" class="byu-component-rendered" has-user="" role="button">
                        <div class="byu-user-wrapper">
                            <div class="no-user slot-wrapper">
                                <div class="user-info-image" aria-label="User Icon"> </div>
                                <span class="text slot-wrapper">
                                    <a slot="login" href="#login">Sign In</a>
                                </span>
                            </div>
                            <div class="has-user">
                                <span class="name slot-wrapper">
                                    <span slot="user-name">Joe Student</span>
                                </span>
                                <div class="user-info-image" aria-label="User Icon"> </div>
                                <span class="logout slot-wrapper">
                                    <a slot="logout" href="#logout">Sign Out</a>
                                </span>
                            </div>
                        </div>
                    </byu-user-info>
                </div>
                <div class="byu-header-search">
                    <byu-search slot="search" action="submit-form" class="byu-component-rendered">
                        <div id="search-form" class="">
                            <div id="search-container" class="">
                                <form action="search.php">
                                    <input name="search" class="__byu-search-selected-input" placeholder="Search" title="Search" type="search">
                                </form>
                            </div>
                            <button id="search-button" type="submit" class="">
                                <img id="search-icon" src="https://cdn.byu.edu/shared-icons/latest/fontawesome/search-white.svg"
                                    alt="Run Search" class=""> </button>
                        </div>
                    </byu-search>
                </div>
            </div>
        </div>
    </div>
    <div class="menu-outer-wrapper">
        <div class="menu-inner-wrapper slot-wrapper needs-width-setting" style="max-width: 1200px;">
            <byu-menu slot="nav" class="byu-component-rendered">
                <nav class="outer-nav slot-container needs-width-setting">
                    <a href="#link1">Link 1</a>
                    <a href="#link2">Link 2</a>
                    <a href="#link3">Link 3</a>
                    <a href="#link4">Link 4</a>
                </nav>
            </byu-menu>
        </div>
    </div>
</byu-header>
```

I've actually simplified that a little. It's not pretty, and Web Components give us this
nice little wrapper behind which we can hide all of this complexity.

## There are Always Trade-offs

Web Components come with their own challenges, though. A year ago, there were still a lot of 
unanswered questions. [Polymer 2](https://www.polymer-project.org/) was still a few months 
from being released, and we weren't enamoured with Polymer's insistence on using the 
never-going-to-be-implemented HTML Imports specification. 
[SkateJS](http://skatejs.netlify.com/) was an option, but we didn't really like the way
Skate was handling templates.

### Opting-out of a Framework

We decided early on that we didn't really need a Web Component framework. Our components
were going to be fairly static - stamp the DOM once, allow for a few small dynamic
things, and be done with it. So, we thought, it wouldn't be that terrible to use the
raw Custom Element APIs, augmented with a small collection of helper functions for doing
common things (like stamping a template to the Shadow DOM).

If we were starting this project now, I think we would be using some form of framework, 
probably Polymer 3 (even though it's not released yet) or SkateJS 5 (with it's nice new 
pluggable renderers). While our setup isn't THAT complicated, we do still re-invent the
wheel in a lot of ways. As I stated on Twitter,

{{< tweet 904902463588839424 >}}

### Singing the Load-Time Blues

That still left the problem of loading our elements. With the HTML Imports spec,
you define your elements in an HTML file, which contains your markup, your styles,
and your JS code, all in once nice happy file. You pop a `<link rel="import">` into
your page, and your components magically load everything they need.

However, we have a lot of skittish developers on campus, and we didn't want to introduce
many new concepts to them (Yes, there are some who would get scared off by something
as simple as a new type of `<link>` tag). We wanted to be able to stick with simple
`<script>` tags for loading. You add into that the fact that nobody but Chrome wanted to
implement HTML Imports, and things were looking pretty dismal for the Polymer 1 and 2 way
of loading things. (Sidenote: I still long for a 
[better solution](https://github.com/w3c/webcomponents/issues/645) to importing
HTML than just 'stick it all in Javascript').

We also wanted our development experience to be nice. If we couldn't just bring in
an HTML document with embedded CSS and JS, we wanted to be able to keep our CSS,
JS, and HTML in separate files, but still have them all be imported with one simple
script tag.

First, we wrote a simple Gulp module that would assemble things for us, but it was
finnicky and relied on comments that were formatted in juuust the right way. That's
when we discovered Webpack, which solved this problem for us. Now, we could have
arbitrary dependency graphs, use modules from NPM, and just generally get things
done without having to write our own bundler. That has worked really well for us and,
as a result, importing our theme component bundle is simple:

```html
<script async src="https://cdn.byu.edu/byu-theme-components/latest/byu-theme-components.min.js"></script>
```

(We'll cover our CDN later).

### Styling the Unstyleable



