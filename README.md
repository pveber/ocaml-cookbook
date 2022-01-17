# An OCaml cookbook

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
