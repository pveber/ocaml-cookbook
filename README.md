# An OCaml cookbook

### What are ocamlc, opam, dune, package, libraries, modules,etc?
### I'm confused with the compilation ecosystem!
### How do I use third-party code?

Let's review the role of each tool and the associated concepts:

- an OCaml source file has extension `.ml`
- each `.ml` file defines a **compilation unit**. You can think of it
  as a mere bag of definitions (functions, types, *etc*). As the name
  suggests, it is the basic granularity manipulated by the compiler.
- at the language level, a compilation unit is represented by a
  **module**. If I have a file `foo.ml` containing a definition of a
  function `f`, I can access the latter from another compilation unit
  by `Foo.f`
- a file `foo.ml` can come with a `foo.mli` file. While `foo.ml`
  provides the *implementation* of the compilation unit, `foo.mli`
  provides a *signature* that is, specifies what's visible from the
  outside of the compilation unit
- the **compiler** is called `ocamlc`, and it transforms text programs
  (files with `.ml`/`.mli` extensions) into compiled code. A compiled
  implementation has extension `.cmo`, while a compiled signature has
  extension `.cmi`. A compiled compilation unit is thus represented by
  two files, one `.cmo` file and one `.cmi` file.
- there's a second compiler, called `ocamlopt` that produces native
  code, while `ocamlc` produces bytecode. Bytecode is portable accross
  machines but not as efficient as native code. `ocamlopt` compiles
  `.ml` files into `.cmx` files (the native code counterpart to
  `.cmo`)
- the compiler (`ocamlc` or `ocamlopt`) also knows how to produce an
  executable from a set of compiled compilation units. Calling the
  compiler directly to produce an executable is perfectly doable (see
  the manual for [bytecode](https://ocaml.org/manual/comp.html) and
  [native](https://ocaml.org/manual/native.html) compilation), but
  there is a more convenient way, using `dune`
- [dune](dune.readthedocs.io/) is a program that helps compiling OCaml
  code into executables (or libraries, see below)
- you provide a description of your project in text files, and `dune`
  figures out the proper way to call the compiler from it.
- `dune` uses a notion of **library**, which was introduced long ago
  by [ocamlfind](https://github.com/ocaml/ocamlfind)
- a library is really nothing more that a set of compilation units
  which is given a name, along with some information
- that information includes options to be passed to the compiler, but
  also dependencies on other libraries. Those dependecies mean that
  when passing the compilation units of the library to the compiler,
  the compilation units of its dependencies should be passed too
- this notion of library is just useful for compilation, it is not
  represented in the language that only knows about modules.
- the compiler itself doesn't know anything either about libraries, it
  only deals with compilation units.
- a library is then just a concept used by `ocamlfind`, `dune` and
  other front-ends to the compiler, to make it possible to specify and
  refer to a group of compilation units that go together
- for instance in `dune`, you can specify that your program depends on
  a library `bar`, and `dune` will look at a conventional location for
  a description of that library, and use it to generate proper calls
  to `ocamlc`/`ocamlopt`
- all that requires that compilation units and library information are
  present at expected locations
- `opam` is a package manager, that is a tool to handle the
  installation of OCaml code, and ensure everything is set to run the
  compiler and allow it to access the installed code
- a **package** as defined by opam is a recipe to install libraries or
  executables
- this recipe includes a lot of information, like a version number of
  the package, contact info, and most importantly dependencies between
  packages (including version constraints)
- opam doesn't really know about libraries as defined by
  `ocamlfind`. It works only at the level of packages.

Let's wrap it up with an example. Suppose I need to use regular
expression. I can use the excellent [re
library](https://github.com/ocaml/ocaml-re). Note that I just used the
word library in a casual sense, let's see what it means precisely: I
will use `opam` to install the [package named
re](https://opam.ocaml.org/packages/re/). This will install at a
proper location a set of libraries called `re`, `re.pcre`, `re.perl`,
*etc*. I can check that running

```sh
$ ocamlfind list | grep "^re"
...
re                  (version: 1.10.3)
re.emacs            (version: 1.10.3)
re.glob             (version: 1.10.3)
re.pcre             (version: 1.10.3)
re.perl             (version: 1.10.3)
re.posix            (version: 1.10.3)
re.str              (version: 1.10.3)
...
```

Let's say I want to use the Perl flavor of regular expressions. In my
`dune` file I will add the `re.perl` library as a dependency (see
`dune`'s
[manual](https://dune.readthedocs.io/en/stable/quick-start.html#building-a-hello-world-program-using-lwt)).
In my program, I can then refer to the module `Re_perl` which
corresponds to a compilation unit included in the library `re.perl`.
When asking `dune` to compile my code, it will notice that the library
`re.perl` depends on the library `re` and will thus invoke `ocamlc` by
providing on the command line the path of compiled compilation units
from `re` along with those from `re.perl`.

More on this:

- https://ocaml.org/learn/tutorials/compiling_ocaml_projects.html
- https://ocaml.org/manual/comp.html
- https://ocaml.org/manual/native.html

### How to represent a vector?

The most immediate approach is to use a `float array`
```ocaml
# let v = [| 1. ; 0. ; 0. |];;
val v : float array = [|1.; 0.; 0.|]
```
It is easy to define common operations:
```ocaml
# let norm v =
    Array.fold v ~init:0. ~f:Float.(fun acc x -> acc + x ** 2.)
    |> Float.sqrt
  ;;
val norm : float array -> float = <fun>

# let dot_product u v =
    Array.fold2_exn u v ~init:0. ~f:(fun acc x y -> acc +. x *. y)
  ;;
val dot_product : float array -> float array -> float = <fun>
```

For dealing with vectors, a `float array` is better than a `float
list`, because the coefficients of a vector often need to be accessed
in arbitrary ways (so-called *random access*). Lists only allow
efficient access to their first element (head), while any coefficient
of an array can be accessed in constant-time. In addition, float
arrays are more compact: consecutive elements are next to each other
in memory, while each element of a list is equipped with a pointer to
the next element.

If you're dealing with 2D or 3D vectors, it can be convenient to use
record types

```ocaml
# type vec2d = {
    x : float ;
    y : float ;
  }
  ;;
type vec2d = { x : float; y : float; }

# let norm v = Float.(sqrt (v.x ** 2. + v.y ** 2.));;
val norm : vec2d -> float = <fun>
```
The [gg library](https://github.com/dbuenzli/gg) provides basic
datatypes and functions for computer graphics.

### How to sum numbers in an array

It's a useful application of `fold`
