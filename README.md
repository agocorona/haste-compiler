Hate
====
[![Build Status](https://travis-ci.org/valderman/haste-compiler.svg?branch=master)](https://travis-ci.org/valderman/haste-compiler.svg?branch=master)

A compiler to generate JavaScript code from Haskell.

It even has a [website](http://haste-lang.org) and a
[mailing list](https://groups.google.com/d/forum/haste-compiler).

Features
--------

* Seamless, type-safe single program framework for client-server communication
* Support for modern web technologies such as WebSockets, WebStorage and Canvas
* Simple JavaScript interoperability
* Generates small, fast programs
* Supports all GHC extensions except Template Haskell
* Uses standard Haskell libraries
* Cabal integration
* Simple, one-step build; no need for error prone Rube Goldberg machines of
  Vagrant, VirtualBox, GHC sources and other black magic
* Concurrency and MVars with Haste.Concurrent
* Unboxed arrays, ByteArrays, StableNames and other low level features
* Low-level DOM base library
* Easy integration with Google's Closure compiler
* Works on Windows, GNU/Linux and Mac OS X


Installation
------------

You have three options for getting Hate: installing from Hackage, from
Github or from one of the pre-built
[binary packages](http://haste-lang.org/#downloads).
In the first two cases, you need to add add Cabal's bin directory, usually
`~/.cabal/bin`, to your `$PATH` if you haven't already done so.
When installing from the Mac, Windows or generic Linux package, you may want
to add `path/to/haste-compiler/bin` to your `$PATH`.
The Debian package takes care of this automatically.

Then, installing the latest stable-ish version from cabal is easy:

    $ cabal install haste-compiler
    $ haste-boot

Building from Github source is equally easy. After checking out the source,
`cd` to the source tree and run:

    $ cabal install
    $ haste-boot --force --local

You should probably run the test suite first though, to verify that everything
is working. To do that, execute `./runtests.sh` in the Hate root directory.
You may also run only a particular test by executing `./runtests.sh NameOfTest`.
The test suite uses the `nodejs` interpreter by default, but this may be
modified by setting the `JS` environment variable as such:
`JS=other-js-interpreter ./runtests.sh`. Other JavaScript interpreters may or
may not work.

Hate has been tested to work on Windows and OSX platforms, but is primarily
developed on GNU/Linux. As such, running on a GNU/Linux platform will likely
get you less bugs.


Portable installation
---------------------

It is possible to install Hate along with its runtime system and base
libraries into a portable directory.
Each user still has their own package database, which makes this handy
for global installations. To do this, check out the source and run:

    $ cabal configure -f portable
    $ cabal build

Building Hate this way yourself is not recommended however, as pre-booted
[binary packages](http://haste-lang.org/#downloads) built this way are
available for your convenience. Why jump through hoops if you don't have to?

A portable installation needs a working GHC install of the same version
that was used to build Hate available on your `$PATH`.


Usage
-----

To compile your Haskell program to a JavaScript blob ready to be included in an
HTML document or run using a command line interpreter:

    $ hastec myprog.hs

This is equivalent to calling ghc --make myprog.hs; Main.main will be called
as soon as the JS blob has finished loading.

You can pass the same flags to hastec as you'd normally pass to GHC:

    $ hastec -O2 -fglasgow-exts myprog.hs

Hate also has its own set of command line arguments. Invoke it with `--help`
to read more about them. In particular `--opt-all`, `--opt-minify`,
`--start` and `--with-js` should be fairly interesting.

If you want your package to compile with both Hate and, say, GHC, you might
want to use the CPP extension for conditional compilation. Hate defines the
preprocessor symbol `__HASTE__` in all modules it compiles.

Hate also comes with wrappers for cabal and ghc-pkg, named haste-inst and
haste-pkg respectively. You can use them to install packages just as you would
with vanilla GHC and cabal:

    $ haste-inst install mtl

This will only work for libraries, however, as installing JavaScript
"executables" on your system doesn't make much sense. You can still use
`haste-inst build` to build your "executables" locally, however.

Finally, you can interact with JavaScript code using the FFI. See
`doc/js-externals.txt` for more information about that.

For more information on how Hate works, see
[the Hate Report](http://haste-lang.org/hastereport.pdf "Hate Report"),
though beware that parts of Hate may have changed quite a bit.

You should also have a look at the documentation and/or source code for
`haste-lib`, which resides in the `libraries/haste-lib` directory, and the
small programs in the `examples` directory, to get started.


Interfacing with JavaScript
---------------------------

When writing programs you will probably want to use some native JavaScript
in your program; bindings to native libraries, for instance. There are two ways
of doing this. You can either use the GHC FFI as described in
`doc/js-externals.txt`, or you can use the Fay-like `ffi` function:

    addTwo :: Int -> Int -> IO Int
    addTwo = ffi "(function(x, y) {return x + y;})"

The `ffi` function is a little bit safer than the GHC FFI in that it enforces
some type invariants on values returned from JS, and is more convenient. It is,
however, quite a bit slower due to its dynamic nature.

If you do not feel comfortable throwing out your entire legacy JavaScript
code base, you can export selected functions from your Hate program and call
them from JavaScript:

fun.hs:

    import Haste.Foreign
    
    fun :: Int -> String -> IO String
    fun n s = return $ "The number is " ++ show n ++ " and the string is " ++ s
    
    main = do
      export "fun" fun

fun.js:

    function mymain() {
      console.log(Haste.fun(42, "hello"));
    }

...then compile with:

    $ hastec '--start=$HASTE_MAIN(); mymain();' --with-js=fun.js fun.hs

`fun.hs` will export the function `fun` when its `main` function is run.
Our JavaScript obviously needs to run after that, so we create our "real" main
function in `fun.js`. Finally, we tell the compiler to start the program by
first executing Hate's `main` function (the `$HASTE_MAIN` gets replaced by
whatever name the compiler chooses for the Hate `main`) and then executing
our own `mymain`.


Effortless type-safe client-server communication
------------------------------------------------

Using the framework from the `Haste.App` module hierarchy, you can easily write
web applications that communicate with a server without having to write a
single line of AJAX/WebSockets/whatever. Best of all: it's completely type
safe.

In essence, you write your web application as a single program - no more forced
separation of your client and server code. You then compile your program once
using Hate and once using GHC, and the two compilers will magically generate
client and server code respectively.

You will need to have the same libraries installed with both Hate and vanilla
GHC (unless you use conditional compilation to get around this).
`haste-compiler` comes bundled with all of `haste-lib`, so you
only need to concern yourself with this if you're using third party libraries.
You will also need a web server, to serve your HTML and JS files; the binary
generated by the native compilation pass only communicates with the client part
using WebSockets and does not serve any files on its own.

Examples of Haste.App in action is available in `examples/haste-app` and
`examples/chatbox`.

For more information about how exactly this works, see this
[draft paper](http://haste-lang.org/haskell14.pdf).


Base library and documentation
------------------------------

You can build your own set of docs for haste-lib by running
`cabal haddock` in the Hate base directory as with any other package.

Or you could just look at
[the online docs](http://hackage.haskell.org/package/haste-compiler).


Reactive web EDSL
-----------------

Fursuit, the reactive EDSL previously shipped together with Hate, had several
serious problems and has now been deprecated. Other, much better, solutions
which work with Hate include [Yampa](http://hackage.haskell.org/package/Yampa),
[elerea](http://hackage.haskell.org/package/elerea) and others.


Libraries
---------

Hate is able to use standard Haskell libraries. However, some primitive
operations are still not implemented which means that any code making use 
of them will give you a compiler warning, then die at runtime with an angry
error. Some libraries also depend on external C code - if you wish to use such
a library, you will need to port the C bits to JavaScript yourself (perhaps
using Emscripten) and link them into your program using `--with-js`.


Why yet another Haskell to JavaScript compiler?
-----------------------------------------------

Existing implementations either produce huge code, require a fair amount of
work to get going, or both. With Hate, the idea is to give you a drop-in
replacement for GHC that generates relatively lean code.


Known issues
------------

* Not all GHC primops are implemented; if you encounter an unimplemented
  primop, please report it together with a small test case that demonstrates
  the problem.

* Template Haskell is still broken.
