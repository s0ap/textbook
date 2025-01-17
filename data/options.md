# Options

Suppose you want to write a function that *usually* returns a value of type
`t`, but *sometimes* returns nothing.  For example, you might want to define a
function `list_max` that returns the maximum value in a list, but there's not a
sensible thing to return on an empty list:

```
let rec list_max = function
  | []   -> ???
  | h::t -> max h (list_max t)
```

There are a couple possibilities to consider:

 - Return `min_int`. But then `list_max` will only work for integers&mdash;
   not floats or other types.

 - Raise an exception.  But then the user of the function has to remember
   to catch the exception.
 
 - In Java, you might return `null`.  OCaml does not have a `null` value. 
   Which is actually a good thing:  null pointer bugs are not fun to debug.

 - In addition to those possibilities, OCaml provides something even better
   called an *option.*  (Haskellers will recognize options as the Maybe monad.)
   
You can think of an option as being like a closed box.  Maybe there's something
inside the box, or maybe box is empty.  We don't know which until we open the box.
If there turns out to be something inside the box when we open it, we can take
that thing out and use it.  Thus, options provide a kind of "maybe type," which
ultimately is a kind of one-of type:  the box is in one of two states, full
or empty.

In `list_max` above, we'd like to metaphorically return a box that's empty
if the list is empty, or a box that contains the maximum element of the list
if the list is non empty.

Here's how we create an option that is like a box with `42` inside it:
```
# Some 42
- : int option = Some 42
```
And here's how we create an option that is like an empty box:
```
# None
- : 'a option
```
The `Some` means there's something inside the box, and it's `42`.  The
`None` means there's nothing inside the box.

As for the types we see above, `t option` is a type for every type `t`, 
much like `t list` is a type for every type `t`. Values of type `t option` 
might contain a value of type `t`, or they might contain nothing.  `None`
has type `'a option` because it's unknown what the type is of the thing inside,
as there isn't anything inside.

You can access the contents of an option value `e` using pattern matching.
Here's a function that extracts an `int` from an option, if there is one inside,
and converts it to a string:
```
# let extract o = 
    match o with 
    | Some i -> string_of_int i 
    | None -> "";;
val extract : int option -> string = <fun> 

# extract (Some 42);;
- : string = "42"

# extract None;;
- : string = "" 
```

Here's how we can write `list_max` with options:
```
let rec list_max = function
  | []   -> None
  | h::t -> begin
      match list_max t with
        | None   -> Some h
        | Some m -> Some (max h m)
      end
```

(The `begin`..`end` wrapping the nested pattern match above is not
strictly required here but is not a bad habit, as it will head off
potential syntax errors in more complicated code.)

In Java, every object reference is implicitly an option.  Either there
is an object inside the reference, or there is nothing there.  That
"nothing" is represented by the value `null`.  Java does not force
programmers to explicitly check for the null case, which leads to null
pointer exceptions.  OCaml options force the programmer to include a
branch in the pattern match for `None`, thus guaranteeing that the
programmer thinks about the right thing to do when there's nothing
there.  So we can think of options as a principled way of eliminating
`null` from the language. Using options is usually considered better
coding practice than raising exceptions, because it forces the caller to
do something sensible in the `None` case.

**Syntax and semantics of options.**

 - `t option` is a type for every type `t`.

 - `None` is a value of type `'a option`.

 - `Some e` is an expression of type `t option` if `e : t`.
    If `e ==> v` then `Some e ==> Some v`

