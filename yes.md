# Introduction

It’s common for mainstream libraries to have unreadable generic signatures and instantiations. We see this everywhere — client or server-side, libraries for any purpose or goal. Here is an example from [TanStack Query](https://tanstack.com/query/latest):

```typescript
export function useQuery<
  TQueryFnData,
  TError,
  TData = TQueryFnData,
  TQueryKey extends QueryKey = QueryKey,
>(
  arg1: TQueryKey | UseQueryOptions<TQueryFnData, TError, TData, TQueryKey>,
  arg2?:
    | QueryFunction<TQueryFnData, TQueryKey>
    | UseQueryOptions<TQueryFnData, TError, TData, TQueryKey>,
  arg3?: UseQueryOptions<TQueryFnData, TError, TData, TQueryKey>,
): UseQueryResult<TData, TError>
```

Many other examples are referenced in #54254, each one more complicated than the next, and I’m sure we’ve encountered these ourselves. One common method of managing these, usually called a “generic bag,” goes something like this:

```typescript
type GenericBag = {
    TypeArg1: unknown
    TypeArg2: unknown
}

declare function doSomething<Bag extends GenericBag>(arg1: Bag["TypeArg1"], arg2: Bag["TypeArg2"]);
```

While I’ve always liked the direction of this method, it’s still a hack — and one that often fails to work, judging by the fact that they’re not a universal construction. 

I can attest to that. I’ve used them in the past, and to this day the way these things behave is still incomprehensible to me – in every instance, finding a configuration that worked was a process of gruesome trial and error, and when I had to deal with several at the same time I often gave up.

Often, for reasons that were unclear to me, I ended up having to declare utility types such as:

```typescript
type GetTypeArg1<Bag extends GenericBag> = Bag extends { TypeArg1: infer TypeArg1 } ? TypeArg1 : never
```

Which, through the problems associated with inferred types, added new complexity to something already bordering on the unmanageable.

But let’s say you successfully finish your construction, add the necessary unit tests, and deploy your package. The story is still far from over — because now, should users of your package step out of line, your monstrous generic creations are thrust upon them as terrifying error messages, describing types that will haunt their waking dreams and nightmares alike.

## A problem statement

I don’t think the problem with such complex generic signatures is something that can be solved using shorthands, tricks, or things like that. Instead, I think what we have is a limitation of the language:

**TypeScript is structural, but type parameters don’t correspond to any structure, and instead are treated as rewriting rules that exist outside the type system.** As such, TypeScript is inherently unable to reason about type parameters.

This same stumbling block hits many different suggestions or proposals that try to do anything with generic parameters, unless it can be understood under the rewriting paradigm. The most that we can hope for is cosmetic improvements. 

While this system has worked in simpler times for simpler types, it’s no longer enough. When simple tools are used to solve complex problems, the result is often unmanageable complexity. **The solution to this complexity is the introduction of more powerful abstractions.**

This proposal is an attempt to do so. To be more specific, the goal of this proposal is **to turn type parameters into structure that TypeScript can reason about.**

## Specific issues

This proposal introduces new, more powerful abstractions that improve the way generic types are used and written. As such, it subsumes many other feature requests – either partially or entirely. Here is an **incomplete list** of issues that are addressed:

| Req    | Name                             | How it’s addressed                                           |
| ------ | -------------------------------- | ------------------------------------------------------------ |
| #54254 | Named type parameters            | Meta types, there is also a short hand                       |
| #26242 | Partial generic parameterization | Meta object inference                                        |
| #17588 | Associated typesI                | Implicit structure, but classes don’t support it directly    |
| #14662 | Generic type display             | More declared entities, fewer params, simpler-looking signatures |
| #42388 | `where` clause                   | Solves the same issues, including “scoped aliases”           |
| #39526 | Overloading generic signatures   | Disjunction meta types                                       |
| #1213  | HKTs                             | Via HKT extension                                            |

I’m sure there are more use cases that I haven’t even thought of.

# The Meta Type System

**The meta type system is the result of applying TypeScript’s structural type system to itself**. It allows us to encapsulate complex generic signatures, construct powerful abstractions, fulfill multiple feature requests with a single type system extension.

The meta type system has two components:

* The **meta object**, a type-level structure that embeds types like normal objects embed values.
* The **meta type**, which is the type of a meta object. 

The meta type system is a unique extension designed specifically for TypeScript, and based on existing TypeScript concepts, rules, and conventions. While it resembles features found in other languages, it’s very much its own thing, and it wouldn’t make sense anywhere else.

We can still make some analogies, though. A good one is to Haskell’s [higher-kinded types](https://en.wikipedia.org/wiki/Kind_(type_theory)). **Where Haskell uses the language of functions to talk about types, here we use the language of objects to do so.**

# Basics

This proposal introduces novel type system constructs that don’t exist anywhere else, so I have to introduce the concept slowly and with a lot of silly examples.

I’m going to look at a specific use case after this part though.

## Meta objects

**A meta object resembles an object value in the world of types.** Where normal objects embed values like `5` and `“hello world”`, meta objects embed **type values** like `string` and `{ id: number, name: string }`. These types don’t form part of the structure of the meta object, but are treated as scalars embedded in its structure.

Meta objects, like most type system constructs, can be given as an expression. Here is an example of one:

```typescript
< 
    Foo := number, 
    Bar := string
>
```

Note the definitional symbol `:=` – we use it because the symbols `:` and `=` are taken, as well as to indicate that the meta type system is involved.

Meta objects can be declared, exported, and imported. Here is how that looks:

```
export meta object Foobar := <
	Foo := number,
	Bar := { id: number }
>
```

Meta objects can also be nested, just like regular objects. Note that the object type `{…}` doesn’t indicate nesting – instead, it indicates an embedded type (that happens to be an object). This is how nesting looks like:

```
<
	Foo := <
		Bar := string
	>,
	Baz := object
>
```

**Like all TypeScript entities, meta objects are purely structural.** They just have a new form of structure which I like to call *meta structure*. Two meta objects with the same meta structure (including embedding the same types, up to equivalence) are equivalent.

Looking at an individual meta object, we can see that it has two sorts of members:

1. Type members
2. Meta members – other meta objects

The meta members form the structure of a meta object, while the type members are embedded like scalar values in that structure. 

To better understand meta objects, let’s compare them to existing structures and patterns in TypeScript code.

### Correspondence 1: Generic type instantiations

One of the reasons behind the `<…>` notation is the correspondence between meta objects and the instantiations of generic types.

Let’s say we have generic type and we instantiate it with a set of type arguments:

```typescript
type Foo<A, B> = [A, B]
type Fooed = Foo<number, string>
```

Then the instantiation correspondence to the meta object:

```typescript
<
	A := number,
	B := string
>
```

In fact, this allows us to define a **natural spread operator for type parameters**. This operator works using key-value semantics rather than sequential semantics. Kind of strange for a spread operator in this context, but I think I prefer it.

```typescript
meta object Types := <
    A := number,
    B := string
>
    
type AlsoFooed = Foo<...Types>
```

### Correspondence 2: Namespaces (and module types)

Take a look at a namespace like this:

```typescript
namespace Example {
	export type A = string
	namespace Inner {
		export type B = string
	}
}
```

This namespace defines several different entities.

1. The value binding `Example`.
2. The type referred to using `typeof Example`.
3. The type structure of `Example`, which allows us to do `Example.Inner` and so on. 

The third entity corresponds to a meta object also named `Example`. 

**In other words, meta objects are like namespaces that only contains exported type declarations.**

This means that meta objects can be used just like namespaces, even though this isn’t the main way you’d use them. 

```typescript
let a: Example.A
```

Something similar applies to module types — they have a hidden meta object component, which is also the set of types and namespaces they export.

### Meta objects and references

Meta objects are value-like constructions in the world of types that describe a rigid and finite tree structure.

However, because types aren’t part of that structure, the types in a meta object can reference other types in the same meta object, or even themselves. This just works like types embedded in a namespace.

```typescript
namespace Illustration {
	type A = string
	type B = {
		value: A
	}
}
```

This means that the following are legal constructions:

```
< 
    A := string, 
    B := { value: A } 
>
<
	A := { value: A } 
>
```

However, these kinds of references can’t be used for the actual structure of a meta object – the `:=` bindings themselves – as it can create infinite structures or even contradictions. While some references can be allowed as a shorthand:

```typescript
< A := string, B := A >
```

Anything that resolves into a cycle isn’t allowed:

```
// COUNTER EXAMPLES
meta object CYCLE := < 
	A := < 
		X := A 
	> 
>
// ERROR: Cycle

meta object MUTUAL_CYCLE := < 
	A := < 
		X := B 
	>, 
	B := < 
		X := A 
	> 
>
// ERROR: Mutual cycle
```

## Meta types

A meta type defines a structural template which some meta objects match and others don’t. The template consists of named **members**, each of which must be constrained somehow. Here is an example of a meta type declaring a member and constraining it via a **meta annotation**.

```
meta type Foo := <
	Bar: type
>
```

This meta annotation says that `Bar` must be a data type (through the special word `type`), rather than a meta object. 

To understand how meta types work, let’s compare them to object types. Here is an object type that’s very similar to `Foo`:

```typescript
type Baz = {
	Bar: unknown;
}
```

It has all sorts of instances, such as:

```
{ Bar: "abc" }
{ Bar: 15 }
{ Bar: () => 5, Also: 200 }
```

As we’ve talked about earlier, meta objects embed types where there used to be values. To get instances of `Foo` we follow the same pattern, just replace all the values with their types:

```
< Bar := string >
< Bar := number >
< 
   Bar := () => number,
    Also := number
    >
```

Here are some non-instances of `Foo`:

```
< A := number >
// ERROR: Must have type variable `Bar`

< 
	Bar := <
		X := string
	>
>
// ERROR: `Bar` must be a type member
```

Meta objects can be nested, meaning they can have other meta object members. Meta annotations can describe these as well – we just need to use another meta type instead of the special token `type`.

```
meta type Foobar := <
	Inner: Foo
>
```

Here are some instances of `Foobar`:

```
< 
	Inner := <
		Bar := string
	>
>
< 
	Inner := <
		Bar := string,
		Also := number
	>
>
< 
	Inner := <
		Bar := string
	>,
	Also := string
>
```

 non instsnxes 



Meta types can also constrain members using a **subtype constraint**. This works just like a subtype constraint in a generic signature. This only makes sense for type members, so the `: type` met annotation is added if it’s not already present:

```
meta type SubtypeExample := <
	Fst extends { 
		readonly value: string 
	}
	Snd: type extends { 
		readonly value: number 
	}
>
```

Here are some instances of this meta type:

```
< 
	Fst := { 
		readonly value: string
        readonly also: number 
    },
	Snd := { 
		readonly value: number 
	}
>
< 
	Fst := { 
		value: string 
	},
	Snd := {
    	readonly value: 100
     }
>

// This also works:
<
	Fst := never,
	Snd := never
>
```

Here are some non-instances:

```
< Fst := string, Snd := never >
// ERROR: Fst must be a subtype of `{ readonly value: string }`

< 
	Fst := <
		X := string
	>,
	Snd := never
>
// ERROR: Fst must be a type, but it was a meta object.

<
	Fst extends { a: string },
	Snd extends { a: number }
>
// ERROR: That's another meta type.

{ Fst: { a: string }, Snd: { a: number }}
// ERROR: That's a data type!
```

So meta types actually have two ways of constraining their members, while regular types have just one. This makes sense because they are a higher-order type system one level above the normal one. They can constrain members on their own level or on the level below.

> Actually, there is more to this: when a meta type places a subtype constraint on a member, it’s placing constraints on its own values. For regular types to do the same thing they would need to phrase constraints in terms of *their* values, which are things like strings and numbers.
>
> It might look something like this:
>
> ```
> // This isn't real or proposed here
> type A = {
> 	foo must|x -> x > 10|
> }
> type B = {
> 	bar must|x -> x.includes("bob")|
> }
> ```
>
> This is a feature called [dependent typing](https://www.fstar-lang.org/tutorial/book/part1/part1_getting_off_the_ground.html#boolean-refinement-types). So if we had that, both meta types and types would be able to place two constraints – a type constraint and a value constraint. 

Constraints in meta types are allowed to reference other members. This works just like in type parameters. Here is an example:

```
<
	Bar: type
	Foo extends Bar
>
```

Here are some instances:

```
< Bar := string, Foo := string >
< Bar := string, Foo := "abc" >
< Bar := {}, Foo := never >
```

Here are some non-instances:

```
< Bar := string, Foo := number>
// ERROR: Expected Foo to be a subtype of string

< A := string >
// ERROR: Must have members Bar, Foo
```

Constraints are also allowed to reference themselves:

```
< LinkedList extends { value: unknown, next: LinkedList | null } >
```

It’s possible to write contradictory or tautological constraints. Although ideally this would be prevented by the current mechanisms for avoiding cycles in similar generic constraints, it’s probably impossible in practice. The language manages to be both undecidable and unsound, after all.

> #### *The abyss*
>
> We could pretend we’re satisfied with what I just said, or we could venture into the abyss. 
>
> The abyss is the alternative universe where we *don’t* try to limit cycles at all. After all, what are types but predicates over a universe of values? And what right do *we*, as humans, have to police the world of well-formed predicates?
>
> In the abyss, we might meet strange denizens:
>
> ```
> < Liar extends (Liar extends true ? false : true) >
> < Truther extends Truther >
> < One extends TheOther, TheOther extends One >
> ```
>
> But shouldn’t we strive to accept all types, even if their forms prove monstrous to us? 

## Using meta types from normal code

### Via type parameters

We can turn a regular type parameters into a meta object parameter by giving it a meta annotation:

```typescript
declare function doSomething<T: MetaType>(): void
```

We can also do something else. You may recall the correspondence between meta objects and generic instantiations. Well, something similar applies between meta types and generic signatures. 

This means that would have rest-like type parameter which is meta annotated, just like a rest parameter that’s type annotated.

```typescript
declare function doSomethingElse<...Args: MetaType>(): void
```

Except that the parameter would have key-value semantics instead of sequential semantics.

### Via namespaces

As we’ve discussed, the type declarations of a namespace are a meta object. As such, we can use meta annotations on them:

```typescript
meta type Foo := < A: type >
namespace Something: Foo {
	export type A = string
}
```

This makes sure the namespace declares the appropriate types.

### Via modules

This is inspired by #420. Module exports can also be meta annotated as part of a module-wide `implements`-like clause. This was never actually implemented, but at least the meta annotation component should be implemented.

```typescript
export: Foo
```

## Inference – doing it and controlling it

Meta types can potentially allow users to partially specify a generic instantiation and have inference complete it. Let’s examine such a situation.

Say we have a generic function as discussed earlier, but we only specify one of the type parameters:

```
generic<A := string>()
```

Under the lens of the meta type system, these kinds of problems become meta type checking problems. We have a meta object `< A := string >` and a meta type `<A: type, B: type>`. The meta object isn’t an instance of the meta type, so instead we assume the user wants us to construct an instance via inference.

`A := string` is already specified, so the meta type is narrowed to `<A := string, B: type>`. If we pick the broadest type for `B`, we get `<A := string, B := unknown>`, which is a valid assignment.

We can use similar logic for more complex inference, but the problem of constructing a meta object instance of a meta type is probably undecidable, so sometimes inference will fail.

We can also have special ways to communicate with the inference process, from the point of view of the API designer – the user who defines the meta type. One method that already exists for doing this is default type parameters:

```
<
	A: type = string
>
```

The `=` symbol here is used to communicate to the inference process, instead of being type information. It’s kind of similar to a decorator.

If you use a default value, you can omit the `: type` annotation because it can be inferred.

## Implicit structure

Implicit structure is structure that belongs to a meta type, which imparts it on the meta object. This is how types work in other languages, like C# and JavaScript – and how they can’t possibly work in TypeScript.

The instances of meta types are meta objects, though, which are very far from the runtime world. On this higher plane of existence we can basically do whatever we want – especially if it’s something really useful. 

And implicit structure is very useful, serving the same function as methods do in OOP. It provides the functionality, while the non-implicit structure forms the internal state of the object.

Here is how it looks like:

```
meta type Foo := <
	Input: type
	Predicate := (x: Input) => boolean
	Array := Input[]
>
```

Implicit structure is defined using the `:=` symbol, the same symbol that is used for the members of a meta object. This is meant to convey that the structure is “fixed”, where the rest of the structure is variable. So if two meta objects have the same non-implicit structure, and they are annotated with the same meta type, they must also have the same implicit structure.

The types can even be mutually recursive, as long as it’s not an obvious cycle.

```
<
	Input extends {a: Other | null}
	Other := Input
>
```

Of course, the implicit structure of different meta types can just be different. The simplest example is a meta type that only has implicit structure:

```
meta type Bar := <
	A := string
	B := number
>
meta type Baz := <
	A := object
	B := boolean
>
```

These look just like meta objects! And that’s because this is the equivalent of the type `{a: 5}` versus the value `{a: 5}`. The type and value look the same because the value is a minimal instance of the type.

### Needs more exploration 

I’m not exactly sure how the implicit structure is applied and how it would work in different situations. Some of the potential issues are:

* What happens if you have two meta types with different implicit structure but the same explicit structure?
* Does a meta object actually get the structure, or is it more like an extension method type of deal?

However, this is actually not how I originally envisioned this feature — it used to work as an equivalence constraint together with meta object inference. The burden of constructing the necessary structure would always fall on instantiation, which should be able to use the constraining expression together with assigning what we call non-implicit structure.

Of course it could also fail to construct the instance and error. 

I’m not sure if basing a feature like this on inference is the best way, but let’s just assume this is how it works until we have something better.

## Shorthands ✨

Some shorthand syntax can be used to make meta types more convenient to use.

### Generic instantiation shorthand ✨

This shorthand is for the following code:

```typescript
declare function generic<A, B>(): void
generic<...<A := string, B := number>>()
```

Essentially we can eliminate the usage of `…` and allow the surrounding `<…>` of the generic instantiation to form a meta object.

```
generic<A := string, B := number>()
```

The usage of `:=` suggests that this isn’t how type parameters are normally treated and that the meta type system is involved.

In this situation, the order of the parameters can’t matter as we’re treating them as members of a meta object.

## Operators and Operations

There are a few operations we can define on meta types and meta objects. 

### Meta objects

Meta objects are value-like constructions in the world of types, and as such support value-like operators.

#### Combination ✨

This is a spread operator for meta objects, but it applies recursively (in contrast to the JS version). It’s similar to `&` in behavior, but meta objects aren’t types.

```typescript
meta object One := <
    A := number
>
meta object Two := <
    B := string
>
meta object Three := <
    ...One,
    ...Two
>
```

An interesting case happens if the two have conflicting structures.

```
meta object One := < A := number >
meta object Two := < A := string >
meta object Three := <
	...One,
	...Two
>
```

From a purist perspective, we might want this to be illegal. After all, then you can say that `{…X, …Y}` always has the structure of both. 

However, if we let the operator behave like the JavaScript equivalent, we would have a powerful method of switching out specific types with other types in a meta object:

```typescript
meta type Func := <
    In: type
    Out: type
    Call := (x: A) => Out
>
type SwitchTheOut<F: Func, Out> := <...F, Out := string>.Call
```



### Meta types

#### Disjunction**✨**

Disjunction can be used between two meta types, but not between a data type and a meta type. So for example something like this is not allowed:

```typescript
// BAD!
string | < A extends string >
```

This, however, is allowed:

```typescript
meta type Foo := 
    | < A extends string, Type := "OneThing" > 
    | < B extends string, C extends string, Type := "Another" >
```

This works like union types work today – it must be disambiguated. The only way to do that is via a conditional involving a shared member of the meta type. So basically reproducing a pattern found in existing TypeScript code, but at the meta level.

```typescript
type GetSomething<F: Foo> = F.Type extends "OneThing" ? F.A : F.B
```

In principle, we could use an actual meta type test, maybe calling the operator for this `instanceof`:

```typescript
// Don't want to support right now
type GetSomething<F: Foo> = Foo instanceof < A extends string > ? F.A : F.B
```

However, I’d like to avoid using meta objects in conditionals like this at this stage.

Regardless, this does open the door to generic signatures with alternative structures. You’d probably need the normal signature to also be variadic, like in this function:

```
declare function doTheFoo<F: Foo>(
	...args: F.Type extends "OneThing" ? [F.A] : [F.A, F.B]
): void
```

#### Conjunction

Conjunction, or the `&` operator, merges the structure and constraints of two meta types to create a third which is a subtype of both.

```typescript
< A extends string > & < B extends number >
```

If the subtypes have different constraints on some member, the constraints can sometimes be merged. This merger needs to be defined separately for each combination of constraints, and some constraints contradict each other. 

```
< A := string> & < A := number >
```

When this happens, the result is always `never`, though in practice it would probably just be an error. 

However, this is different:

```
meta type Both = string & < A: type >

function blah<X: Both>(x: X) {}
```

This situation is intriguing and I have conflicting thoughts about whether it should work and how it would act.

