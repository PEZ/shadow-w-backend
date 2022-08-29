# nREPL – Starting shadow-cljs from lein or clojure

This is a tiny example project for setting up development using [nREPL](https://nrepl.org/) and [shadow-cljs](https://github.com/thheller/shadow-cljs). The [ClojureScript](https://clojurescript.org) app is rendered using [Reagent](https://reagent-project.github.io/).

By default the project uses [deps.edn](https://clojure.org/guides/deps_and_cli) for dependencies. There is a [Leiningen](https://leiningen.org/) setup as well.

## Install npm dependencies

```
npm i
```

(Or `yarn`, whatever you fancy.)

## Using a Clojure editor

If you are using something like [Calva](https://calva.io/) or [CIDER](https://cider.mx/), just jack in to the project as a **shadow-cljs** project type, and hack away.


## Starting the project w/o an editor

The following instructions are for working with the project without an editor. You probably wouldn't do that. The point is mostly to demystify what the editors are doing when starting and attaching to a shadow-cljs project. So, you could say it is for educational purposes. Since it's pretty quick, why not try it out?

You can start this project using either `shadow-cljs`, `clojure`, or `lein`. As you are doing this to maybe learn something, please consider trying all three options.

### With the shadow-cljs npm executable

```
npx shadow-cljs -d cider/cider-nrepl:0.28.5 clj-repl
...
shadow.user=>
```

NB: If you use `shadow-cljs` it will in turn use `clojure` to start Clojure and the nREPL server. (Or `lein` if you edit `shadow-cljs.edn` and change this.)

### With deps/`clojure`

To start the project and the nREPL server with `clojure`, in one terminal:

```
% clojure -Sdeps '{:deps {nrepl/nrepl {:mvn/version,"1.0.0"},cider/cider-nrepl {:mvn/version,"0.28.5"}}}'  -m nrepl.cmdline --middleware "[cider.nrepl/cider-middleware,shadow.cljs.devtools.server.nrepl/middleware]"
```

Attach to the nREPL server in another terminal:

```
% clojure -Sdeps '{:deps {reply/reply {:mvn/version "0.5.1"}}}' -M -m reply.main --attach `< .nrepl-port` 
...
user=> 
```

Or, since the `deps.edn` file specifies the `nrepl` dependency:

```
% clj -M -m nrepl.cmdline -c --port `< .nrepl-port`
...
user=> 
```

### With Leiningen

To start the project and the nREPL server with Leiningen, in one terminal:

```
lein update-in :dependencies conj '[nrepl,"1.0.0"]' -- update-in :dependencies conj '[cider/cider-nrepl,"0.28.5"]' -- update-in :plugins conj '[cider/cider-nrepl,"0.28.5"]' -- update-in '[:repl-options,:nrepl-middleware]' conj '[cider.nrepl/cider-middleware,shadow.cljs.devtools.server.nrepl/middleware]' -- repl :headless
```

**NB: This doesn't properly inject the shadow-cljs nREPL middleware for some reason.** You'll need to add this to `project.clj`:

```clojure
  :repl-options {:nrepl-middleware [shadow.cljs.devtools.server.nrepl/middleware]}
```

Attach to the nREPL server in another terminal:

```
% lein repl :connect
...
main.server=> 
```

## Starting shadow-cljs

(Not needed if if you started the REPL using the shadow-cljs executable.)

``` clojure
user=> (do (require 'shadow.cljs.devtools.server) (shadow.cljs.devtools.server/start!))
shadow-cljs - HTTP server available at http://localhost:8700
shadow-cljs - server version: 2.19.9 running at http://localhost:9632
shadow-cljs - nREPL server started on port 57770
:shadow.cljs.devtools.server/started
user=> 
```

(Never mind the second nREPL server started here.)

At this point the shadow-cljs dev browser is running and serving on port 8700, but your ClojureScript app is not running. It is not even compiled. As you'll see if you visit http://localhost:8700 with your browser.

## Start the shadow-cljs watcher

(The `require` is not needed if you started the REPL with the shadow-cljs executable.)

``` clojure
user=> (do (require 'shadow.cljs.devtools.api) (shadow.cljs.devtools.api/watch :app))
[:app] Configuring build.
[:app] Compiling ...
[:app] Build completed. (170 files, 0 compiled, 0 warnings, 1.35s)
:watching
user=> 
```

## Start the ClojureScript app

Open http://localhost:8700 in a web browser. (Or refresh if you opened it in an earlier step.)

Confirm that the shadow-cljs watcher is running by editing some hiccup in `src/main/core.cljs` and save the file.

## Attach to the ClojureScript REPL

(The `require` is not needed if you started the REPL with the shadow-cljs executable.)

``` clojure
user=> (do (require 'shadow.cljs.devtools.api) (shadow.cljs.devtools.api/nrepl-select :app))
To quit, type: :cljs/quit
[:selected :app]
cljs.user=>
```

(If you instead of `[:selected :app]` see `:missing-nrepl-middleware`, you are probably using Leiningen and missed a note above.)

Confirm that your REPL is attached to the app in the browser:

```clojure
cljs.user=> (js/alert "BOOM!")
```

(You need to dismiss the alert in the browser to get the prompt back.)

Something more interesting could be to update the app state, maybe?

```clojure
cljs.user=> (load-file "src/main/core.cljs")
[]
cljs.user=> (in-ns 'main.core)
nil
main.core=> (swap! app-state update :text str " UPDATED")
{:text "Hello world! UPDATED"}
```

(Check the app in the browser.)

## Notes

If you use this project as a base for something, then edit `.gitignore` and remove `package-lock.json`.

The reason the Leiningen project file is not including the shadow nrepl middleware configuration is that I think [the failure to inject via the command line is a bug in Leiningen](https://codeberg.org/leiningen/leiningen/issues/10) and that it will eventually be fixed.