# Function Application & Pattern Matching Demo

## Forward

Module type `Fwd_S` specifies three functions, `int`, `bool` and `string`, that each take an argument of the corresponding type and return a string. *I.e., "string-ify" functions.*

```ocaml
module type Fwd_S = sig
  val int : int -> string
  val bool : bool -> string
  val string : string -> string
end
```

Module `A_Fwd` ...

```ocaml
module A_Fwd : Fwd_S = struct
  let int (x : int) : string = Printf.sprintf "%i" x
  let bool (x : bool) : string = Printf.sprintf "%b" x
  let string (x : string) : string = x
end
```

Module `B_Fwd` ...

```ocaml
module B_Fwd : Fwd_S = struct
  let int : int -> string = Printf.sprintf "%i"
  let bool : bool -> string = Printf.sprintf "%b"
  let string : string -> string = fun x -> x
end
```

## Main Matter

### Variant Type

Type `t` is a variant type with constructors corresponding to `int`, `bool` and `string`. We will use this type to demonstrate different methods of function appilcation and pattern matching.

```ocaml
type t =
  | Int of int
  | Bool of bool
  | String of string
```

### Basics

Module type `F_S` specifies function `val f : t -> string`. Therefore, any module of type `F_S` must provide a function `f` that can handle the three variants of `t` (`t.Int`, `t.Bool` and `t.String`) and return a `string`.

```ocaml
module type F_S = sig
  val f : t -> string
end
```

Module `A_F` implements `F_S.f` in a straightforward way. It takes the first argument of type `t` and binds it to `x`. Next, it pattern matches on `x` to determine which variant of `t` the value of `x` is. This pattern matching provides us the opportunity to determine what to do for each variation; in this case we must provide a function for each that maps it to a `string`. *(If only we had functions that could do this...)* Notice that module `A_F` is a *functor* which takes a module argument of type `Fwd_S` and binds it to `X`. By the signature of `Fwd_S` we know that `X` implements a function for converting each of the cases of `t` to a string; *how convenient!* It just remains for us match-up the corresponding function in `X` to the pattern-matched cases of `x`.

```ocaml
module A_F (X : Fwd_S) : F_S = struct
  let f (x : t) : string =
    match x with
    | Int (x : int) -> X.int x
    | Bool (x : bool) -> X.bool x
    | String (x : string) -> X.string x
  ;;
end
```

Module `B_F` implements `F_S.f` is similar to `A_F`, except that it's definition of `F_S.f` skips the binding of the first argument and proceeds directly into the pattern matching, via use of the `function | _ -> ...` keyword/pattern. Notice that this is reflected in the "return signature" of `B_F.f` with respect to `A_F.f`, as `B_F.f` returns a function that takes a `t` which returns a `string`, while `A_F.f` is itself a function that takes a `t` and returns a `string`. Both fulfill the signature specified by `F_S.f`.

```ocaml
module B_F (X : Fwd_S) : F_S = struct
  let f : t -> string = function
    | Int x -> X.int x
    | Bool x -> X.bool x
    | String x -> X.string x
  ;;
end
```

### Continued

```ocaml
module type Foo_S = sig
  val foo : t -> unit -> string
end

module A (X : F_S) : Foo_S = struct
  let foo : t -> unit -> string = fun (x : t) -> fun () -> X.f x
end

module B (X : F_S) : Foo_S = struct
  let foo : t -> unit -> string = function (x : t) -> fun () -> X.f x
end

module C (X : F_S) : Foo_S = struct
  let foo (x : t) : unit -> string = fun () -> X.f x
end

module D (X : F_S) : Foo_S = struct
  let foo (x : t) : unit -> string = function () -> X.f x
end

module E (X : F_S) : Foo_S = struct
  let foo (x : t) () : string = X.f x
end
```
