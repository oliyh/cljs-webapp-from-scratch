# cljs-webapp-from-scratch

You may have never heard of Clojure, much less be aware of its use for web development.
I have worked with several programming languages over my career but since discovering Clojure
I find myself reaching for it no matter the problem I have to solve. I'd like to introduce it
to you today and show how your next web application could be your first steps into the
happy world of a Clojure developer.

In this article we will cover some of the reasons why ClojureScript is a natural fit for web development,
particularly with React via a library called Reagent. We'll use a build tool called shadow-cljs and write
a basic interactive web application.

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

There are many excellent resources for learning Clojure[^1] but for now a crash course:

```clj
:span                   ;; a keyword
[:div :span]            ;; a vector with keywords in it
{:width 100}            ;; a map, with a key `:width` and a value `100`

(defn double-me [n]     ;; a function called `double-me` that takes one argument
  (* 2 n))              ;; returning `2 * n`

(double-me 4)           ;; an example of calling the `double-me` function with `4`
```

Congratulations, you've just learned nearly the entire syntax used by Clojure!

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

You're now ready to start shadow-cljs, which will compile your code, serve it to the browser and
watch the filesystem for any changes you make:
```
npx shadow-cljs watch app
```

You can open http://localhost:8020 in your browser to see the skeleton.

The following diagram should help you understand what's going on here:
![shadow-cljs processes](/shadow-cljs.png?raw=true)

### Building a page

The source code generated from the quick start is under `quickstart/src/main/starter/browser.cljs`.

It contains some functions which are hooks to get you going with your development lifecycle:
- `init` (called once when the page loads, which calls `start` as well)
- `start` (called after new code has been loaded)
- `stop` (called before new code is loaded)

Let's try pushing a dom element into the page.
Add the following to the `start` function so it looks like this:

```clj
(defn ^:dev/after-load start []
  (.appendChild (js/document.getElementById "app")
                (js/document.createTextNode "Hello, world")))
```

When you save this source file with your changes, shadow-cljs will compile
and push the updated code into your browser to be evaluated.
You will see "Hello, world" appear before your eyes!

## Rendering framework

Our example above serves as a sighter to orient ourselves in our new surroundings.
The browser, the DOM and Javascript are all there as before but we have now ascended to a higher plane,
and our horizons have become broader.

[React](https://reactjs.org/) is a hugely popular library for rendering pages.
Its functional, immutable approach is a natural fit for Clojure and there are several
Clojure wrappers that provide a more idiomatic API[^2].

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

We will need to restart the `npx shadow-cljs watch app` to pick up the new dependency.

We can now write our first reagent component. It takes the form of a simple Clojure function
that returns a Clojure data structure representing HTML known as [Hiccup](https://github.com/weavejester/hiccup)
after the library that popularised the format. It is much more concise than HTML and plays much more
nicely with a structural code editor (e.g. paredit in emacs).

```clj
(defn- hello-world []
  [:ul
   [:li "Hello"]
   [:li {:style {:color "red"}} "World!"]])
```

This represents the following HTML:

```
<ul>
  <li>Hello</li>
  <li style="color: red;">World!</li>
</ul>
```

We need to require the `reagent.dom` namespace:

```clj
(ns starter.browser
  (:require [reagent.dom :as rd]))
```

And then use it to mount our component into the DOM by changing the `start` function to look like this:

```clj
(defn ^:dev/after-load start []
  (rd/render [hello-world] (js/document.getElementById "app")))
```

Once you have saved the file this list should now be rendered in your browser.
We now have the rendering basics

### Interactivity

Things start to get interesting when we decide to add some interactivity. After all, if your page
is static then you don't need Javascript at all. Before we implement anything, however, I'd like
to introduce you to one of Clojure's superpowers: the REPL.

#### REPL

The REPL is a prompt where you can evaluate Clojure expressions. This is a powerful tool
which helps you explore and trial solutions, reducing your feedback loop and making you more productive.

It looks like this:

```
cljs.user=> (+ 1 1)
2
```

Without further delay, let's start a REPL so you too can experience the mind-blowing thrill of
adding one number to another. As we're building a web page, it will be useful
to use a browser to perform the evaluation in our REPL. It's easy to start one; in a new terminal run:

```
npx shadow-cljs cljs-repl app
```

When you opened the page http://localhost:8020 in your browser, it included
some code injected by shadow-cljs to open a websocket back to the server.

When you type commands into this REPL, they are sent via this websocket
to be evaluated in the browser, with results returned to your REPL.

Try out the addition example above. How do we know it was evaluated in the browser? Try this next:
```clj
(js/alert "Hello, world")
```

You should see an alert dialogue in your browser,
as if you had typed `alert("Hello, world")` in the browser's developer console.

Pro tip: press `Ctrl+D` when you want to exit the REPL.

Let us now use this tool to help with implementing an interactive state in our app.

#### Atoms

You may be wondering how we are going to implement mutable state when Clojure's data structures are immutable.
Clojure is a pragmatic language, so there are actually mutable constructs in it, but they are carefully marked
so it's clear that you are dealing with something special.

An atom is a mutable reference to an immutable value. The reference can be mutated using `swap!` or `reset!`
(the `!` being conventional notation for a mutation) and the atom can be dereferenced to get the value using `@` or `deref`:

```clj
(def counter (atom 0))

@counter ;; => 0

(swap! counter inc)
@counter ;; => 1
```

We are going to use an atom in our application to store some state, but we are going to use reagent's version of an atom.
It has the same interface as Clojure's atom, but it has a secret superpower - when it changes, it can tell React to re-draw the DOM.

We need the reagent core namespace:
```clj
(ns starter.browser
  (:require [reagent.core :as r]
            [reagent.dom :as rd]))
```

And we can create an initial state:
```clj
(defonce state (r/atom {:items ["Hello" "World!"]}))
```

We can write a new component with an input box to allow us to add items into the state:
```clj
(defn- new-item []
  [:input
   {:type "text"
    :placeholder "Enter a new item"
    :on-key-down (fn [e]
                   (when (= "Enter" (.-key e))
                     (swap! state update :items conj (.. e -target -value))))}])
```

And finally change our `hello-world` component to list out the items from the state:
```clj
(defn- hello-world []
  [:div
   [new-item]
   [:ul (map (fn [item]
               [:li {:key item} item])
             (:items @state))]])
```

You will notice that `hello-world` also includes our `new-item` component as a child.

This now renders as below, and when we type a new item into the input and press enter, it joins the list!
![shadow-cljs processes](/interaction.webm.mp4)


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
[^1]: [Official Clojure guide](https://clojure.org/guides/learn/syntax), [Eric Normand's collection of learning resources](https://ericnormand.me/mini-guide/the-ultimate-guide-to-learning-clojure-for-free)
[^2]: Other wrappers for React include [helix](https://github.com/lilactown/helix), [rum](https://github.com/tonsky/rum) and [uix](https://github.com/roman01la/uix)
