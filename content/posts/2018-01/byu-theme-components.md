---
title: "Bringing Order to Web Design Chaos"
description: "Or, how I learned to stop worrying and love Web Components"
canonical_url: https://www.thatjoemoore.com/posts/2018-01/byu-theme-components/
date: 2018-02-08T21:27:42-07:00
published: true
tags: [web components, design]
---
_In my maturation as a Software Engineer, I've found that stories of the problems and solutions that other engineers have encountered to be invaluable tools for improving and learning. So, I'm going to start sharing some of my code stories. This is the first of (hopefully) many._

This is the story of an interesting engineering effort I've participated in over the last year. It's been a really fun side project for me (~20 hours a week at the start, ~5 hours a week since then), which has allowed me to experiment with some fun new technologies and implement some cool stuff. It centers on an effort to bring a simple, unified theme to all of our many, many web properties that can share code between any CMS or frontend framework.

# Where We Started

I work for [Brigham Young University](https://www.byu.edu), a large, private University.

Universities are a unique environment. In a university like ours, you have a number of large colleges and schools, all of which have a lot of freedom to do what they want. Historically, one way in which this freedom has manifested itself is in Web design, which has lead to some very...inconsistent designs.

For example, this time last year, a designer I work with looked at the home pages of all of our top-level colleges and schools, and found that they all had completely different designs. They used different logos, layouts, shades of our school colors (navy blue and white), and, in one extreme case, didn't use our school colors at all. Many of these pages were loosely based on a design that had been published in 2013. This particular design was never well documented  and the reference implementation wasn't portable across platforms, so we saw a lot of divergence and changes as time went on.

I don't mean to shame any particular schools, because most of their designs actually looked quite nice, but here's a ~~horrifying~~ fun picture to show what I'm talking about:

{{< figure src="byu-sites.png" title="See anything wrong here?" >}} 

After hiring an outside firm to conduct an audit of our web presence, it was decided that it was time to try and bring some brand uniformity to the University. An initiative was launched in our web community to design and implement a simple, modern, unobtrusive standard theme and to encourage developers and designers across campus to converge on this theme.

I'm not going to talk about the design part, because I'm not a designer. While I was asked to attend the design meetings, it was as an engineer who would be asked to develop a reference implementation of the theme. I should note, however, that it was decided to only require a small header (including search and user sign-in), simple navigation, and a footer. Everything else was up to individual sites to design, though we've also been working on providing them with off-the-shelf styles and components so that every site doesn't have to redesign common elements.

# Our Goals

Once our design group hammered out a design that everyone could work with, it was time for our engineering group to implement things. While the design group worked, we had one department that stepped up in a big way and developed a Drupal module that allowed them to rapidly iterate on the design across several of their sites, so the designers could get feedback on  what worked in the real world and what didn't. However, for long-term use, we decided that we wanted a solution that gave us three things:

* [ ] Cross-platform
* [ ] Simple
* [ ] Fixable

## Cross-Platform

We have a wide variety of web hosting platforms across campus, ranging from simple static sites, to CMSes like Drupal and Wordpress, to complex single-page apps written in Angular, React, and other frameworks. We needed a solution that would allow a small team to make our theme work consistently across all platforms. Ideally, this would mean having one single codebase for everything.

## Simple

We have a wide variety of skill sets on campus, ranging from seasoned, 30-year veterans to CS students who got a part-time job maintaining a departmental site. We needed a solution that was simple and easy to consume, even if the developer doesn't understand everything that's going on - if you can't reliably copy and paste from examples, it would be too complicated.

## Fixable

One of the problems we had run into in the past was that, as the theme was improved, only new sites would adopt the improvements. Because every site kept their own copy of the theme code, there was no good way to distribute updates and fixes.

# Web Components to the Rescue

A few of us in the group had been playing with Web Components, and the v1 version of the specifications had just been implemented in Chrome. After looking at several proposals on how to reach our goals, we decided that Web Components would give us a lot of what we were looking for.

Web Components solved a lot of our problems. Because they're a platform-level primitive, we didn't have to worry as much about integration - they can (theoretically) integrate with anything that can work with the DOM.

To put this in perspective, it took a couple of student programmers who hadn't seen our code before about 30 minutes to implement a Wordpress theme that was based on our Web Components. My favorite demonstration of the ease with which we could integrate came when I was doing a presentation about the theme to the campus organization I work for. As a live demo, I took one of our really old, nasty pages and retrofitted it to use the new theme, right there in front of everyone. It took about 10 minutes. Granted, I had practiced this beforehand, but it was still pretty fun to show just how easy it could be.

Beyond the ability to write them out to the DOM, Shadow DOM gave us some really nice style isolation, so we could be certain that none of the site's styles could accidentally mess up our themed components, and none of our internal styles would leak out to the outside world.

There have been some hiccups on the way, but our CMSes have had a pretty easy integration story, and we've got Single-Page Apps in production that are using React (16+), Angular (2+), VueJS, and others. There are a handful of SaaS tools doing some truly awful things to the DOM that we haven't gotten working yet, but for the most part, it's been pretty uneventful.

__Goal Check In:__

* [x] Cross-platform
* [ ] Simple
* [ ] Fixable

Web Components also promised to solve some of our complexity problems. If you're familiar with Bootstrap, you know how, in order to use their styles, you have to structure your markup in just the right way, with lots of nested `div`s and special classes. It's really easy to get lost in all of that structure, especially since the design we were given had some very large layout differences between mobile and desktop views. The concept of slots in Shadow DOM gave us a really easy way to hide the markup complexity from the consumer, while still allowing them to plug in all of their site-specific things.

(If you're unfamiliar with Shadow DOM, slots are a lot like doing component composition with `children` in React or `<ng-content>` in Angular)

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

At first glance, there's a lot going on here. However, most of the structural complexity is hidden, and only the things that are specific to a given site are included.

To give you an idea of how much structure has been left out, here's what this looks like after being rendered by the Shady DOM polyfill, which combines all of the content into one DOM tree:

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

I've actually simplified that a little. It's not pretty, and Web Components give us this nice little wrapper behind which we can hide all of this complexity.

__Goal Check In:__

* [x] Cross-platform
* [x] Simple
* [ ] Fixable

# Dynamic Distribution

(This is the part I'm really proud of)

Once we had a working set of components, we needed some way to distribute them to developers on campus. We considered using some sort of package manager, like NPM, but we quickly determined that it would not fit our needs. While it would give us a simple way to push out updates, there would be no way to guarantee that sites would see the update. We have sites that will go years without seeing an update, and, in the event of an important bug, we wanted to be able to have even those sites get the update. In addition, with the variety of platforms we needed to support, it would be hard to converge on a single package manager that would be easy for everyone to use.

The solution we converged on was to build a central CDN and have all sites load resources from it. This has turned out to be a great idea. We built a process that could respond to webhook notifications from Github and push code changes to the CDN. My office has been aggressively pursuing a move to Amazon Web Services, so we build the CDN using Amazon S3, Cloudfront, and CodeBuild.

Our CDN's secret sauce is its assembler. When we make any pushes to any of our Github repositories, an AWS Lambda function receives a webhook notification and kicks off a build running on AWS CodeBuild. We use CodeBuild because a) we don't have to run any servers for it and b) it allows us to execute basically anything that can run in a docker container. The assembler looks at a list of repositories that are included in the CDN, then uses the Github API to find all of the branches and tags that have changed or been created since the last assembler run. It pulls down the necessary files, then copies the required files into our S3 bucket.

In order to provide automatic updates, we include the concept of 'aliases' in the CDN. Aliases use semver parsing of Git tags to create aliases like `latest` (always the highest version number), `1.x.x` (gets all minor and bugfix releases), and `1.1.x` (gets all bugfix releases for the 1.1 major release). We also include an `unstable` alias, which points to whatever is currently on master. Users can also stick to a specific release by providing an explicit version number. This allows us to provide automatic updates to all most sites, while giving more control to those who desire it.

When a consumer wants to use something from our CDN, they construct a URL like this:

```
https://cdn.byu.edu/{library name}/{version or alias}/{file name}
```

So, the URL for the main Javascript for the always up-to-date version our theme components is `https://cdn.byu.edu/byu-theme-components/latest/byu-theme-components.min.js`. Getting the current latest version and turning off auto-updates would have a URL of `https://cdn.byu.edu/byu-theme-components/1.2.3/byu-theme-components.min.js`.

We want to maximize the cacheability of all of our assets, but this can complicate the rapid rollout of new releases. We've found a good compromise by assuming that tags will never change (which is usually true). So, we serve anything coming from a specific tag with cache headers that indicate that it is cacheable forever (`Cache-Control: immutable, public, max-age=31557600, s-maxage=31557600`). Then, when a user requests an alias (like `latest`), we send back a 302 redirect to the specific tag and set a cache expiry on the redirect to 1 hour. This gives us a good balance between having updates roll out often and keeping useful things in the user's cache. The redirects happen quickly enough and transfer little enough data to the user that they don't add much overhead, and we keep the vast majority of our assets cached in the browser forever (well, a year in most browsers, but that might as well be forever, right? I mean, it's longer than the lifetime of most Javascript frameworks).

__Goal Check In:__

* [x] Cross-platform
* [x] Simple
* [x] Fixable

# Additional Challenges

There are always problems that you don't forsee going into a project (or even some that you forsee, but decide not to tackle until later). Here are some of the ones we faced.

## Opting-out of a Framework

When we started working on this a year ago, [Polymer 2](https://www.polymer-project.org/) was still a few months from being released, and we weren't enamoured with Polymer's insistence on using the never-going-to-be-implemented HTML Imports specification. [SkateJS](http://skatejs.netlify.com/) was an option, but we didn't really like the way Skate was handling templates.

Faced with these options, we decided early on that we didn't really need a Web Component framework. Our components were going to be fairly static - stamp the DOM once, allow for a few small dynamic things, and be done with it. So, we thought, it wouldn't be that terrible to use the raw Custom Element APIs, augmented with a small collection of helper functions for doing common things (like stamping a template to the Shadow DOM).

If we were starting this project now, I think we would be using some form of framework, probably Polymer 3 (even though it's not released yet) or SkateJS 5 (with its nice new pluggable renderers). While our setup isn't THAT complicated, we do still re-invent the wheel in a lot of ways.

{{< tweet 904902463588839424 >}}

## Singing the Load-Time Blues

That still left the problem of loading our elements. With the HTML Imports spec, you define your elements in an HTML file, which contains your markup, your styles, and your JS code, all in once nice happy file. You pop a `<link rel="import">` into your page, and your components magically load everything they need.

However, we have a lot of skittish developers on campus, and we didn't want to introduce many new concepts to them (Yes, there are some who would get scared off by something as simple as a new type of `<link>` tag). We wanted to be able to stick with simple `<script>` tags for loading. You add into that the fact that nobody but Chrome wanted to implement HTML Imports, and things were looking pretty dismal for the Polymer 1 and 2 way of loading things. (Sidenote: I still long for a [better solution](https://github.com/w3c/webcomponents/issues/645) to importing HTML than just 'stick it all in Javascript').

We also wanted our development experience to be nice. If we couldn't just bring in an HTML document with embedded CSS and JS, we wanted to be able to keep our CSS, JS, and HTML in separate files, but still have them all be imported with one simple script tag.

First, we wrote a simple Gulp module that would assemble things for us, but it was finnicky and relied on comments that were formatted in juuust the right way. That's when we discovered Webpack, which solved this problem for us. Now, we could have arbitrary dependency graphs, use modules from NPM, and just generally get things done without having to write our own bundler. That has worked really well for us and, as a result, importing our theme component bundle is simple:

```html
<script async src="https://cdn.byu.edu/byu-theme-components/latest/byu-theme-components.min.js"></script>
```

## Styling the Unstyleable

Shadow DOM is great for keeping our template CSS contained, but what about styling the pieces that the consumer hands us in our slots? The whole point of this project is the styling!

Shadow DOM does provide an answer for styling slotted content, but it's a limited one. I can make use of the `::slotted` pseudo-element to style them:

```html
<style>
  .my-slot-wrapper ::slotted(*) {
    color: navy;
  }
</style>
<div class="my-slot-wrapper">
    <slot></slot>
</div>
```

You'll notice that `::slotted` takes in another selector. That selector can be any CSS selector, with one limitation: it can only be one level deep. You can't have a selector like `::slotted(div > a)`, it can only be one level deep.

Most of the time, this limitation isn't a problem, but we quickly ran into two major issues: styling links and styling inputs.

As soon as we published a beta version of our theme components, we saw people adding links to their site titles that were nested inside of `<h1>`: 

```html
<byu-header>
  <h1 slot="site-title"><a href="my-site.byu.edu">My Site</a></h1>
</byu-header>
```

Normally, nested text elements wouldn't cause any problems for us - we set `color`, `font-size`, and `font-family` on the parent element, and it cascades down. Links, however, are special. Every user-agent stylesheet I've seen applies their own `color` and `text-decoration` to links, and doesn't inherit those styles from the parent. If the `<a>` is the element with the `slot` attribute, this isn't a problem - the styles get directly applied to it. However, when the link is nested inside another element, we can't style it from within the Shadow DOM.

The other problem we ran into was styling search inputs. We were aiming for compatibility across a wide range of frameworks and CMSes, and each one has their own ways of wrapping inputs that we couldn't get rid of. However, we needed to make sure that their inputs got styled properly. One solution could have been to present our own styled input and use Javascript to keep the two up-to-date with each other, but we felt that this approach had a lot of potential to confuse developers. For example, it would make implementing something like an autocomplete popup very, very difficult.

So, we needed to be able to have this:

```html
<byu-search>
  <div class="my-cms-wrapper">
    <input type="search" />
  </div>
</byu-search>
```

get styled just like:

```html
<byu-search>
  <input type="search" />
</byu-search>
```

The solution to both of these problems turned out to be something we were already familiar with: external CSS stylesheets. We added a stylesheet (we called it the 'extra' styles) containing special selectors to handle these and other corner cases, and we have developers include both the stylesheet and the Javascript in their pages:

```html
<link rel="stylesheet" href="https://cdn.byu.edu/byu-theme-components/latest/byu-theme-components.min.css">
<script async src="https://cdn.byu.edu/byu-theme-components/latest/byu-theme-components.min.js"></script>
```

Developers were already used to including pairs of CSS and Javascript files (like with jQuery UI), so we weren't introducing anything new to them.

Because we were using custom element names, the selectors in the 'extra' stylesheet can be very simple, yet specific:

```css
byu-header > [slot="site-title"] a {
    text-decoration: none;
    color: white;
}
```

While we try to avoid selectors that are more than two layers deep, having the outer two layers (`byu-header > [slot="site-title"]`) gives us pretty good scoping, so our styles don't bleed over and affect the styles on the page.  There's still the possibility that some stylesheet later on the page will do this something like

```css
a {
    color: green;
}
```

We decided that, since we can't solve everything, we won't try to solve this problem. After all, this is just an instance of CSS behaving as expected, so developers should be able to figure out why the site title is suddenly green all on their own (well, with the help of their browser's devtools, at least).

## &lt;blink&gt; Considered Harmful _(Flash of Unstyled Content)_

One of the pitfalls to using JS to render parts of a UI is the Flash of Unstyled Content. This is a common problem across frontend frameworks and isn't specific to Web Components, though I hope that proposals like [Declarative Shadow DOM](https://github.com/w3c/webcomponents/blob/gh-pages/proposals/Declarative-Shadow-DOM.md) can help reduce the impact of it when using Web Components.

Typically, the strategy for overcoming this is to embed some CSS in the initial HTML that will only apply until the framework has loaded and starts rendering things. We don't have a good way to have people embed styles and still preserve our goal of easily pushing out updates, so we've compromised and embed the Flash of Unstyled Content styles in our 'extra' styles. This CSS file is much smaller and parses much quicker than our JS, so we can get a very good experience from it. Once the components have stamped their template, they apply the `byu-component-rendered` class to themselves, so all we need to do in our styles is look for components that don't have the class:

```css
byu-header:not(.byu-component-rendered) {
    /* nasty fallback styles here */
}
```

Since our CSS gets resolved synchronously, this allows our pages to look almost-right while we wait for the JS to finish its work. This leads to practically seamless page transitions, especially once the browser has cached the CSS.

# Future Work

One thing we want to add in the future is client-side analytics. While we can gather a fair amount of information from our server logs, we can't see some critical things, like how many sites are using different configuration options and component combinations. We're looking at using `navigator.sendBeacon` to send light-weight analytics back to our servers with information like the build of the code that has been loaded, which components actually get used, different features of each component that are being used, etc. We don't want to rely on an external analytics site, because we don't want to add the concerns about load time and privacy that come with them to all of our sites. If individual sites want to use a full-featured analytics tool, they'll have that option without anything we do interfering with that.

Another thing we want to add is automated tests.  There aren't a lot of logical tests we can do, since there isn't a lot of logic in our components, but we are really looking forward to adding tests that use visual diffing tools like [Pixelmatch](https://github.com/mapbox/pixelmatch) to detect when we've made changes that alter the styling of our elements. The goal will be to have a step in our CDN assembler that runs the tests and doesn't deploy the new code if they fail.

# Summary

This has been a really fun project to work on, forcing us to tackle a lot of interesting challenges. I'm really proud of the solutions we've come up with, as well as the way that a group of developers from different departments and of different skill levels all came together to come up with a solution that works well for everyone. While our code isn't perfect, and we haven't really had a chance to bring it all in line with a single style, I feel like we've done a pretty good job.

For those who would like to take a look at our code, it is all part of the [BYU Web Github Organization](https://github.com/byuweb/) and is licensed with Apache 2.0.

A few quick links:

* [Our main theme components](https://github.com/byuweb/byu-theme-components)
    * [A gallery of the theme components](http://2017-components-demo.cdn.byu.edu/)
* [Our CDN source](https://github.com/byuweb/web-cdn)
* [Drupal](https://github.com/byuweb/byu2017_d7) and [Wordpress](https://github.com/byuweb/WordPress) CMS themes that leverage the Web components.

