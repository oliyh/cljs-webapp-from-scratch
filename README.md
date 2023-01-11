# cljs-webapp-from-scratch

> Todo something jaunty

Welcome, intrepid explorer, to ...

## Why ClojureScript?

Clojure has compelling reasons to choose it for any kind of development, on both the server and the front end.
The language itself is functional and immutability is the default idiom, immediately removing an entire class of issues
relating to mutability from your code and headspace. Its core library, data structures and literals are concise and
powerful.

On the server it will run on the JVM and interop with Java allows it to benefit from the Java ecosystem. In the browser
(Clojure compiles to Javascript too, using [ClojureScript](https://github.com/clojure/clojurescript)) interop with Javascript libraries gives the same advantages.
Clojure itself has an active and [happy](https://www.computerworld.com/article/2693998/clojure-developers-are-the-happiest-developers.html)
community building wonderful software with and for Clojure.

One of our favourite features is the REPL - Read, Eval, Print, Loop - an interactive environment in which you can execute
your code, greatly reducing feedback loops for interative development and encouraging exploration and experimentation.

## Build tool

> Is this next para too much?

One of the things I appreciate about Clojure is that it hits the sweet spot when it comes to choices. The community
is big enough that you always have a couple of options when it comes to doing something, but small and focussed enough
that there isn't a profusion of competing libaries or tools all with different strengths and weaknesses that make it
hard to choose between them.

So too with ClojureScript build tools. The ClojureScript compiler itself is considered pretty low level - it will turn
Clojure into Javascript, but not much more. A handful of build tools have emerged over the last ten years which support
incremental compilation as you develop, launching REPLs, running tests and browser hot-reloading.

- [lein-cljsbuild](https://github.com/emezeske/lein-cljsbuild)
- [figwheel-main](https://figwheel.org/)
- [shadow-cljs](https://github.com/thheller/shadow-cljs)

In this article we shall be using shadow-cljs, which has had the most recent development, enjoys community funding
and has the best support for using libraries from NPM - in other words, arguably the most solid choice today.

### shadow-cljs quickstart

Yet another excellent reason to choose shadow-cljs is the comprehensive documentation.
Among these is the [browser quick start](https://github.com/shadow-cljs/quickstart-browser) which gets you to
a running setup on a skeleton project with just a few commands.

Clone the project and install dependencies:

```
git clone https://github.com/shadow-cljs/quickstart-browser.git quickstart
cd quickstart
npm install
```

> How about an SVG diagram here with the server, app build, REPL and browser all arranged
> with the commands that run them to show how it all fits together and what their jobs are?


You're now ready to start shadow-cljs, which will compile your code, push it into the browser and
watch the filesystem for any changes you make:
```
npx shadow-cljs watch app
```

You can open http://localhost:8020 in your browser to see the skeleton.

### REPL

We can now start a REPL. As we're building a web page, it will be useful
to use a browser to perform the evaluation in our REPL. It's easy to start one; in a new terminal run:

```
npx shadow-cljs cljs-repl app
```

When you opened the page http://localhost:8020 in your browser, it included
some code injected by shadow-cljs to open a websocket back to the server.

When you type commands into this REPL, they are sent via this websocket
to be evaluated in the browser, with results returned to your REPL.

Let's try it out:

```
cljs.user=> (+ 1 1)
2
```

How do we know it was evaluated in the browser? Try this:
```
(js/alert "Hello, world")
```

You should see an alert dialogue in your browser,
as if you had typed `alert("Hello, world")` in the browser's developer console.

Press `Ctrl+D` when you want to exit the REPL.

### Building a page

The source code generated from the quick start is under `quickstart/src/main/starter/browser.cljs`.

It contains some functions which are hooks to get you going with your development lifecycle:
- `init` (called once when the page loads, which calls `start` as well)
- `start` (called after new code has been loaded)
- `stop` (called before new code is loaded)

Let's try pushing a dom element into the page.
Add the following to the `start` function so it looks like this:

```
(defn ^:dev/after-load start []
  (.appendChild (js/document.getElementById "app")
                (js/document.createTextNode "Hello, world")))
```

When you save this source file with your changes, shadow-cljs will compile
and push the updated code into your browser to be evaluated.
You will see "Hello world" appear before your eyes!

## Rendering framework

Our example above serves as a sighter to orient ourselves in our new surroundings.
The browser, the DOM and Javascript are all there as before but we have now ascended to a higher plane,
and our horizons have become broader.

[React](https://reactjs.org/) is a hugely popular library for rendering pages.
Its functional, immutable approach is a natural fit for Clojure and there are several
Clojure wrappers that provide a more idiomatic API[^1].

A safe choice for us here is [reagent](https://github.com/reagent-project/reagent).
One fun fact demonstrating the affinity between React and Clojure is that despite being a wrapper,
Reagent can perform faster than plain React because Clojure's immutable data structures can be compared
more efficiently than Javascript objects, resulting in faster decisions about when to re-render a component.

### Reagent

Add reagent to the `:dependencies` key in `shadow-cljs.edn`:

```
 :dependencies
 [[reagent "1.1.1"]]
```

> Do we need to restart anything or will it work like magic? Test this

We can now write our first reagent component. It takes the form of a simple Clojure function
that returns a Clojure data structure representing HTML known as [Hiccup](https://github.com/weavejester/hiccup)
after the library that popularised the format. It is much more concise than HTML and plays much more
nicely with a structural code editor (e.g. paredit in emacs).

```
(defn- hello-world []
  [:ul
   [:li "Hello"]
   [:li {:style {:color "red"}} "World!"]])
```

And then mount this component into the DOM by changing the `start` function to look like this:
```
(defn ^:dev/after-load start []
  (rd/render [hello-world] (js/document.getElementById "app")))
```

You will now

### Evaluation of reagent

> Is this an appendix?

- Very mature, defacto standard
- Nice API
- Reagent can be faster than React because of ClojureScript's immutable data structures - it's cheaper
to detect if props have changed than Javascript's deep equality
- No good support for hooks
- Stuck on old react?

## State management
- Local ratoms
- React state stuff - not available yet with reagent?
- re-frame for next level

## Beyond

> Are these further articles, will they make it too long?

- Advanced build for deployment
- Building a multi-page app
- IDE integration? Only for a nicer repl, really
- Tests
  - Unit - built in page in shadow
  - webdriver stuff
  - devcards + kamera

## Footnotes
[^1]: Other wrappers for React include [helix](https://github.com/lilactown/helix),
  [rum](https://github.com/tonsky/rum) and [uix](https://github.com/roman01la/uix)
