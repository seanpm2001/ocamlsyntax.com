---
title: "OCaml Syntax Cheatsheet"
---

Fan favorites
{.label}

```ocaml {.ml}
let succ ?x:(y : int = 0) () = y + 1

let foo : type a. a My_gadt.t -> unit = fun x -> ignore x

type t = { f : 'a. 'a list -> int }
```

Mutually recursive modules
{.label}

```ocaml {.mli}
module rec Foo : sig
  type t = Bar of Bar.t | Nil
end and Bar : sig
  type t = Foo of Foo.t | Nil
end
```
```ocaml {.ml, .captioned}
module rec Foo : sig
  type t = Bar of Bar.t | Nil
end = struct
  type t = Bar of Bar.t | Nil
end and Bar : sig
  type t = Foo of Foo.t | Nil
end = struct
  type t = Foo of Foo.t | Nil
end
```

Recursive module definitions always require explicit signatures.
{.caption}

For loop
{.label}

```ocaml {.ml, .captioned}
let sum = ref 0 in
for i = 1 to 3 do
    sum := !sum + i
done;
assert (!sum = 6)
```

Note that the range is inclusive on both ends.
{.caption}

Universally quantified record field
{.label}

```ocaml {.mli}
type t = { f : 'a. 'a list -> int }
```
```ocaml {.ml}
type t = { f : 'a. 'a list -> int }
```

# First-Class Modules

First-class module parameter (module-style)
{.label}

```ocaml {.mli}
val min : (module Comparable.S with type t = 'a) -> 'a -> 'a -> 'a
```
```ocaml {.ml}
let min (type a)
  (module Compare : Comparable.S with type t = a)
  (x : a) (y : a) =
  if Compare.( < ) x y then x else y
```

Named first-class module parameter (module-style)
{.label}

```ocaml {.mli}
val min : compare:(module Comparable.S with type t = 'a) -> 'a -> 'a -> 'a
```
```ocaml {.ml}
let min (type a)
  ~compare:(module Compare : Comparable.S with type t = a)
  (x : a) (y : a) =
  if Compare.( < ) x y then x else y
```

As far as I can tell there is no way to express an optional parameter in this way, even if you provide a default.
{.caption}

First-class module parameter (value-style)
{.label}

```ocaml {.mli}
val min : (module Comparable.S with type t = 'a) -> 'a -> 'a -> 'a
```
```ocaml {.ml}
let min (type a)
  (compare : (module Comparable.S with type t = a))
  (x : a) (y : a) =
  let module Compare = (val compare) in
  if Compare.( < ) x y then x else y
```

Named first-class module parameter (value-style)
{.label}

```ocaml {.mli}
val min : compare:(module Comparable.S with type t = 'a) -> 'a -> 'a -> 'a
```
```ocaml {.ml}
let min (type a)
  ~(compare : (module Comparable.S with type t = a))
  (x : a) (y : a) =
  let module Compare = (val compare) in
  if Compare.( < ) x y then x else y
```

Named optional first-class module parameter with default (value-style)
{.label}

```ocaml {.mli}
val min : ?compare:(module Comparable.S with type t = 'a) -> 'a -> 'a -> 'a
```
```ocaml {.ml}
let min (type a)
  ?(compare : (module Comparable.S with type t = a) = (module struct
    type t = a
    let (<) _ _ = false
    (* etc *)
  end))
  (x : a) (y : a) =
  let module Compare = (val compare) in
  if Compare.( < ) x y then x else y
```

The inline module is only necessary if you need to refer to type variables from the signature. For simpler module types, you can provide a named module as the default with `?(m : S = (module M))`.
{.caption}

First-class module argument
{.label}
```ocaml {.ml}
min (module Int) 1 2
```

# Function Signatures

Annotated return type
{.label}

```ocaml {.mli}
val succ : int -> int
```
```ocaml {.ml}
let succ x : int = x + 1
```

Annotated positional argument
{.label}

```ocaml {.mli}
val succ : int -> int
```
```ocaml {.ml}
let succ (x : int) = x + 1
```

Annotated return type and positional argument
{.label}

```ocaml {.mli}
val succ : int -> int
```
```ocaml {.ml}
let succ (x : int) : int = x + 1
```

Named argument
{.label}

```ocaml {.mli}
val succ : x:int -> int
```
```ocaml {.ml}
let succ ~x = x + 1
```

Renamed argument
{.label}

```ocaml {.mli}
val succ : x:int -> int
```
```ocaml {.ml}
let succ ~x:y = y + 1
```

Named argument with annotated return type
{.label}

```ocaml {.mli}
val succ : x:int -> int
```
```ocaml {.captioned, .ml}
let succ ~x : int = x + 1
```

Note that the space between the identifier and the colon is significant to
disambiguate this from a renamed argument.
{.caption}

Renamed argument with annotated return type
{.label}

```ocaml {.mli}
val succ : x:int -> int
```
```ocaml {.captioned, .ml}
let succ ~x:y : int = y + 1
```

The space between the identifier and the second colon is no longer required, as
there is no more ambiguity.
{.caption}

Annotated named argument
{.label}

```ocaml {.mli}
val succ : x:int -> int
```
```ocaml {.ml}
let succ ~(x : int) = x + 1
```

Annotated renamed argument
{.label}

```ocaml {.mli}
val succ : x:int -> int
```
```ocaml {.ml}
let succ ~x:(y : int) = y + 1
```

Optional argument
{.label}

```ocaml {.mli}
val succ : ?x:int -> unit -> int option
```
```ocaml {.captioned, .ml}
let succ ?x () = Option.map ~f:(fun n -> n + 1) x
```

Note that optional arguments without subsequent positional arguments will generate a compiler warning, so we add a final unit argument to get around that. [See here for more information.][warning-16]
{.caption}

Optional argument with a default value
{.label}

```ocaml {.mli}
val succ : ?x:int -> unit -> int
```
```ocaml {.ml}
let succ ?(x = 0) () = x + 1
```

Annotated optional argument
{.label}

```ocaml {.mli}
val succ : ?x:int -> unit -> int option
```
```ocaml {.ml}
let succ ?(x : int option) () = Option.map ~f:(fun n -> n + 1) x
```

Annotated optional argument with a default value
{.label}

```ocaml {.mli}
val succ : ?x:int -> unit -> int
```
```ocaml {.ml}
let succ ?(x : int = 0) () = x + 1
```

Renamed annotated optional argument with a default value
{.label}

```ocaml {.mli}
val succ : ?x:int -> unit -> int
```
```ocaml {.ml}
let succ ?x:(y : int = 0) () = y + 1
```

Explicit passing of an optional argument
{.label}
```ocaml {.ml}
let succ ?(x : int = 0) () = x + 1
let one = succ ?x:None ()
let two = succ ?x:(Some one) ()
let negative_succ ?(x : int option) () = -(succ ?x ())
```

Locally abstract type (monomorphic)
{.label}

```ocaml {.mli}
val foo : 'a My_gadt.t -> unit
```
```ocaml {.captioned, .ml}
let foo (type a) (x : a My_gadt.t) = ignore x
```

You will often see type variables introduced like this so that a GADT's phantom
can vary over different branches of a match statement. [See here for more information.][monomorphic]
{.caption}

Locally abstract type (polymorphic)
{.label}

```ocaml {.mli}
val foo : 'a My_gadt.t -> unit
```
```ocaml {.captioned, .ml}
let foo : type a. a My_gadt.t -> unit =
  fun x -> ignore x
```

This is useful when you have a recursive GADT where the components of a value
have a different phantom type than the value
itself. [See here for more information.][polymorphic]
{.caption}

[warning-16]: https://ocaml.org/docs/labels#warning-this-optional-argument-cannot-be-erased
[monomorphic]: https://ocaml.org/manual/locallyabstract.html
[polymorphic]: https://v2.ocaml.org/manual/locallyabstract.html#p:polymorpic-locally-abstract
