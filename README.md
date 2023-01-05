# cljs-webapp-from-scratch

## Why ClojureScript?
- Functional, immutable, concise
- Community, happy developers
- Interop
- Identical to Clojure, write once run anywhere
- REPL

## Build tool
cljsbuild, figwheel, shadow

### shadow-cljs quickstart
https://github.com/thheller/shadow-cljs


Browser quick start:
https://github.com/shadow-cljs/quickstart-browser

Clone the project and install dependencies:

```
git clone https://github.com/shadow-cljs/quickstart-browser.git quickstart
cd quickstart
npm install
```

Then start the shadow-cljs server:
```
npx shadow-cljs server
```

You're now ready to build the app itself:
```
npx shadow-cljs watch app
```

You can open http://localhost:8020 in your browser to see the skeleton.

### REPL

We can now start a REPL. As we're building a web page, it will be useful
to use a browser to perform the evaluation in our REPL. It's easy to start one:

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

Press `Ctrl+D` to exit the REPL.

### Building a page

The source code is under `quickstart/src/main/starter/browser.cljs`.

Let's try pushing a dom element into the page:

```
// todo make this just use the text node for brevity?
  (let [p (js/document.createElement "p")
        text (js/document.createTextNode "Hello world")
        app (js/document.getElementById "app")]
    (.appendChild p text)
    (.appendChild app p))
```

When you save this source file with your changes, shadow-cljs will compile
and push the updated code into your browser to be evaluated.

## Rendering framework

Many options here - om, helix, preact, reagent

### Reagent

Add reagent to the `:dependencies` key in `shadow-cljs.edn`:

```
 :dependencies
 [[reagent "1.1.1"]]
```

Restart the shadow-cljs server you started with `npx shadow-cljs server` and the shadow-cljs compiler with ``

We can now write a reagent component:
// todo talk about hiccup syntax

```
(defn- hello-world []
  [:ul
   [:li "Hello"]
   [:li "World!"]])
```

Now we can mount this component into the DOM:
```
(defn ^:dev/after-load start []
  (rd/render [hello-world] (js/document.getElementById "app")))
```

### Evaluation
- Very mature, defacto standard
- Nice API
- No good support for hooks
- Stuck on old react?

## State management
- Local ratoms
- React state stuff - not available yet with reagent?
- re-frame for next level

## Beyond
- Advanced build for deployment
- Building a multi-page app
- IDE integration? Only for a nicer repl, really
- Tests
  - Unit - built in page in shadow
  - webdriver stuff
  - devcards + kamera
