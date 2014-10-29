# SML

## Conceptual ways to build new Types
3 ways to create a compound type:
* "Each-of" : A compound type t describes values that contain each of value of type t1,t2,... and tn. e.g. Records, 
        tuples.
* "One-of": A compuond type t describes values that contain a value of one of the types t1,t2,... or tn. e.g. *int option* is a simple example: A value of this type either contains an int or it does not. Java enumerator.
* "Self-reference": A compund type t may refer to itself in its definition in order to describe recusive data structure
           like list and tree.

### Datatype bindings
The following defines a new type of one-fo type. It defines the constructors (TwoInts, Str, Pizza) and constructor arguments (int*int, string and NONE)
```SML
datatype mytype = TwoInts of int*int
                | Str of string
                | Pizza
                
(* above define 3 constructor functions *)
TwoInts = fn int*int -> mytype
Str = fn string -> mytype
Pizza = val mytype
```

## Currying: (named after Haskell Curry who invent this)
If a function ```fn x*y => z``, then curreing is to have a function take the first comceptual argument ```x``` and return
another function that takes the second conceptual argument ```y``  and return ```z``` so on. The function with only partial conceptual arguments called *partial application*
```SML
val sorted = fn x=> fn y=> fn z=> z>y andalso y>=x
(((sort 4)5)6) = true
(* parenthis is optional *)
sort 4 5 6 = true

(* the following is called partial application *)
sort 4 
sort 4 5

(* alternative way to define currying, separating conceptual arguments by spaces rather than anonymous functions *)
fun sorted x y z = z>y and also y>=x

```

### Partial application vs val-binding
```SML
fun longest_string_helper (f:int*int->bool) (cs:string list) =
(*fun longest_string_helper f cs = *)
    List.foldl (fn (s,acc) => if f(String.size(s), String.size(acc)) then s else acc) "" cs

(* val-binding *)
val longest_string3 = fn cs => longest_string_helper (fn (x,y)=>x>y) cs

(* partial application *)
fun longest_string4 cs = longest_string_helper (fn (x,y)=>x>=y) cs
```

## Closure
A Function value has two pars, the *code* for the function and the *environment that was current when we created the 
function*. These two parts do really form a "pair". When evalates the code part, you will using the environment part.

This "pair" is called a *function closure• or just *closure*. The code itself can have *free variables* (variables that
are nto bound inside the code so they need to be bound by some outer environment), the closure carries with it an 
environment that provides all these bindings. So the closure overall is "closed" -- it has everything it needs to 
produce a function result given a function argument.

```SML
val x=1 
fun f y = x+y
val x=2
val y=3
(* val z = 6 *)
val z=f(x+y) 
```

### Lexical Scope
The body of a function is evaluated in the environment where the funciton is **defined**, not the environment where
the function is called.

### Combining functions
```SML
fun sqrt_of_abs i = Math.sqrt(Real.fromInt (abs i))

(* composition with operator "o" is associate-to-right *)
fun sqrt_of_abs i = (Math.sqrt o Real.fromInt o abs) i

(* pipleline oeprator *)
fun sqrt_of_abs i = i |> abs |> Real.fromInt |> Math.sqrt
```

### Using closure for callbacks
In UI libraries often there are listeners to different events, and the library has no idea about what the listener
want to do. It just call the listener, and when called, each listener can response by using their local variables,
which is perfect for closure.

### Abstract data types




## Returning functions

```SML

fun increment x = x+1
val increment = fn x=>x+1

(* following using an anonymous function y=>3*y *)
fun triple_n_times(n,x) = ntimes((fn y=>3*y), n, x))

fun double_or_triple f= 
if f7
then fn x=> 2*x
else fn x=> 3*x
```

## Mutation via ML References
In ML, most things really cannot be mutated. But Mutation is ok in some settings. A key approach in functional 
programming is to use it when "updating the state of something so all users of that state can see a change has
occured" is the natual way to model your computation.

One good way to think about a reference is as a record with one field where that field can be updated with the **:=**
operator.
```SML
val x=ref 0
val y = !x+1 (* y=1 *)
val _ = x := (!x) +2 (* the content of the reference x refers to is now 3 *)
val z = !x+1 (* z=4 *)
```