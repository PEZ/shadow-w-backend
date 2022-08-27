# nREPL – Starting shadow-cljs from lein or clojure

This is a minimal project for setting up a project for development using nREPL and shadow-cljs. By default i uses deps.edn for dependencies, but there is a Leiningen setup as well.

## npm dependencies

```
npm i
```

(Or `yarn`, whatever you fancy.)

## Starting the project

### deps
To start the project with `clojure`:

```
clojure -Sdeps '{:deps {nrepl/nrepl {:mvn/version,"1.0.0"},cider/cider-nrepl {:mvn/version,"0.28.5"}}}'  -m nrepl.cmdline --middleware "[cider.nrepl/cider-middleware,shadow.cljs.devtools.server.nrepl/middleware]"
```

### Leiningen

To start it with `lein`:

**NB: This does not work, for some reason yet to be figured out**

```
lein update-in :dependencies conj '[nrepl,"1.0.0"]' -- update-in :dependencies conj '[cider/cider-nrepl,"0.28.5"]' -- update-in :plugins conj '[cider/cider-nrepl,"0.28.5"]' -- update-in '[:repl-options,:nrepl-middleware]' conj '[cider.nrepl/cider-middleware,shadow.cljs.devtools.server.nrepl/middleware]' -- repl :headless
```

## Using:

To get to the cljs prompt:

```
% lein repl :connect
Connecting to nREPL at 127.0.0.1:57709
REPL-y 0.5.1, nREPL 1.0.0
Clojure 1.11.1
OpenJDK 64-Bit Server VM 18.0.1+10
    Docs: (doc function-name-here)
          (find-doc "part-of-name-here")
  Source: (source function-name-here)
 Javadoc: (javadoc java-object-or-class-here)
    Exit: Control+D or (exit) or (quit)
 Results: Stored in vars *1, *2, *3, an exception in *e

user=> (do (require 'shadow.cljs.devtools.server) (shadow.cljs.devtools.server/start!) (require 'shadow.cljs.devtools.api) (shadow.cljs.devtools.api/watch :app))
shadow-cljs - HTTP server available at http://localhost:8701
shadow-cljs - server version: 2.19.9 running at http://localhost:9632
shadow-cljs - nREPL server started on port 57770
[:app] Configuring build.
[:app] Compiling ...
[:app] Build completed. (170 files, 92 compiled, 0 warnings, 4.11s)
:watching
user=> (do (require 'shadow.cljs.devtools.api) (shadow.cljs.devtools.api/nrepl-select :app))
To quit, type: :cljs/quit
[:selected :app]
cljs.user=> 
```

You can also use

```
npx shadow-cljs -d cider/cider-nrepl:0.28.5 watch :app
```