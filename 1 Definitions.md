# 1 Definitions

## Meta objects

Meta objects, like most type system constructs, can be declared or given as an expression. Here is a simple example:

```typescript
< 
    Foo := number, 
    Bar := string
>
```

**A meta object resembles an object value in the world of types.** Where normal objects embed values like `5` and `“hello world”`, meta objects embed **type values** like `string` and `{ id: number, name: string }`.

Meta objects can be declared, exported, and imported. Here is how that looks:

```
export meta object Foobar := <
	Foo := number,
	Bar := string
>
```

Meta objects can also be nested, just like regular objects.

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

In fact, this allows us to define a **natural spread operator for type parameters**. This operator works using key-value semantics rather than sequential semantics. Kind of strange for a spread operator in this context, but let’s just go with it.

```typescript
meta object Types := <
    A := number,
    B := string
>
    
type AlsoFooed = Foo<...Types>
```

Actually, this is one way in which Meta Types restructure complex generic types – by turning complicated instantiations into declared entities with their own structure.

### Correspondence 2: Namespaces

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
meta object MUTUAL_CYCLE := < 
	A := < 
		X := B 
	>, 
	B := < 
		X := A 
	> 
>
// COUNTER EXAMPLES
```

## Meta types

A meta type defines a structural template which some meta objects match and others don’t. The template consists of named **members**, each of which must be constrained somehow. Here is an example of a meta type declaring a member and constraining it via a **meta annotation**.

```
meta type Foo := <
	Bar: type
>
```

This meta annotation says that `Bar` must be a data type, rather than a meta object. 

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
< Bar := () => number, Also := number >
```

Here are some non-instances of `Foo`:

```
// !!! COUNTER EXAMPLES !!!

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

Meta types can also constrain members using a **subtype constraint**. This functions just like a subtype constraint in a generic signature. This only makes sense for type members, so the `: type` met annotation is added if it’s not already present:

```
meta type SubtypeExample := <
	Fst extends { readonly value: string },
	Snd: type extends { readonly value: number }
>
```

Here are some instances of this meta type:

```
< 
	Fst := { readonly value: string, readonly also: number },
	Snd := { readonly value: number }
>
< 
	Fst := { value: string },
	Snd := { readonly value: 100 }
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
// Expected Foo to be a subtype of string

< A := string >
// Must have members Bar, Foo
```

Constraints are also allowed to reference themselves:

```
< LinkedList extends { value: unknown, next: LinkedList | null } >
```

It’s possible to write contradictory or tautological constraints. Although ideally this would be prevented by the current mechanisms for avoiding cycles in similar generic constraints, in practice doing so might be impossible due to undecidability and unsoundness considerations.

#### *The dark side*

We could pretend we’re satisfied with what I just said, or we could venture into the abyss.

You see, in the abyss, we don’t even try to limit these cycles. 

After all, what are types but predicates over a universe of values? And what right do *we*, as humans, have to police the world of well-formed predicates?

In the abyss, we might meet strange denizens:

```
< Liar extends (Liar extends true ? false : true) >
< Truther extends Truther >
< One extends TheOther, TheOther extends One >
```

But shouldn’t we strive to accept all types, even should their forms prove monstrous to us? 

## Bringing it together

Meta types are integrated into the rest of the language via type parameters. We can turn a regular type parameters into a meta object parameter by giving it a meta annotation:

```typescript
declare function doSomething<T: MetaType>(): void
```

We can also do something else. You may recall the correspondence between meta objects and generic instantiations. Well, something similar applies between meta types and generic signatures. This means that would have rest-like type parameter which is meta annotated, just like a rest parameter that’s type annotated.

```typescript
declare function doSomethingElse<...Args: MetaType>(): void
```

Except that the parameter would have key-value semantics instead of sequential semantics.

## Implicit structure

Implicit structure is structure that belongs to a meta type, which imparts it on the meta object. This is how types work in other languages, like C# and JavaScript – and how they can’t possibly work in TypeScript.

The instances of meta types are meta objects, though, which are very far from the runtime world. On this higher plane of existence we can basically do whatever we want – especially if it’s something really useful.

Here is how it looks like:

```
meta type Foo := <
	Input: type
	Predicate := (x: Input) => boolean
>
```

Implicit structure is defined using the `:=` operator, the same symbol that is used for the members of a meta object. This is meant to convey that the structure is “fixed”, where the rest of the structure is variable. So if two meta objects have the same non-implicit structure, and they are annotated with the same meta type, they must also have the same implicit structure.

I’m pretty sure it would still work if the types were mutually recursive, at least as long as there isn’t an obvious cycle:

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

### Much is unknown

I’m not exactly sure how the implicit structure is applied and how it would work in different situations. What I originally had was an equivalence constraint combined with type inference. That acted in broadly the same way, but I quickly realized it’s actually one of the most important features of meta types, and it should be phrased that way.

Some of the potential issues are:

* What happens if you have two meta types with different implicit structure but the same explicit structure?
* Does a meta object actually get the structure, or is it more like an extension method type of deal?

When it was an equivalence constraint, inference would just try to construct the type based on its definition at the point of call – or fail to do so, which would probably result in an instantiation error. I’m not sure if that’s the best way though.









that all instances of a meta object will have the same implicit structure if they have the same explicit structure.

1. Ignoring the implicit aspect, It’s an equivalence constraint.
2. But with this implicit aspect, it becomes additional functionality (in the form of type expression) that `HasImplicitStructure` provides to its instances.

This can be likened to the way a type class extends a data type with functionality. Two instances of a meta type can differ in implicit structure only if they differ in explicit structure.

This means that implicit structure can only be derived from the other members of the meta type, though it can also just be constant.

f

Note that in many respects, this meta type resembles a higher-kinded type, something like:

```
type Predicate<SomeType> = (x: SomeType) => boolean
```



1. If we ignore its implicit nature, it becomes an **equivalence constraint**.
2. 

**This is different from how structure works in TypeScript.** Structure is something that values possess whether or not they are connected to types. However, meta types can have implicit structure because both the type and its instances are part of the type system. There is no separation, unlike the separation between type definitions and runtime code.

However, implicit structure is always dependent on and derived from non-implicit structure. As such, implicit structure can only differ between meta objects if their non-implicit structure differs. This means that we only need to compute implicit structure once for every 

Instead, implicit structure works more like casting does in a language like C#. It’s a conversion, and 

This process of applying implicit structure – and potentially removing it – is something that will need to be expanded on further. 

ThImplicit structure is structure that is added into a meta object imp

This kind of structure extends instances of the meta type with additional structure at the point of conformation – but it can also be specified explicitly.

In practice, there are two ways to view static structure:

1. As a type member constrained using a strict **equivalence constraint**.
2. As structure implicitly mixed into meta objects during the point of conformation. 

The two are essentially the same because of the following rule governing implicit structure:

> If two instances of a meta type have different static structure, then they have different non-static structure.

Static structure is the result of side-effects free and memoizable computation that happens on a purely type system level. It’s the structural analog of Scala’s trait system, but embedded in higher-order types.

I think the ability to express implicit structure makes meta types really special 



Implicit members are declared using the `:=` symbol, the same as in meta objects. In many contexts they act as a strict **equivalence constraint**. Meaning that, for a given combination of 

This is possible because both the metatype and its structure are entirely within the type system.

Implicit structure is derived from the variable members of a meta type, so that instances that have identical explicit structure have the same implicit structure. However, the relationship between the two can be complex.

Implicit structure is still part of the structure of a meta object. In fact, implicit structure can be made explicit as long as 

Implicit structure is defined using the `:=` definitional operator. Here is an example:

```
meta type HasImplicitStructure := <
	Foo: type
	Predicate := (x: Foo) => boolean
	Array := Foo[]
>
```







To understand how this would work, let’s imagine a similar feature applied to regular types – something that, again, can’t be done in TypeScript.

```typescript
// THIS IS NOT POSSIBLE

type LinkedList = {
    // regular member
	value: unknown
    
    // regular member
    next: LinkedList | null
    
    // implicit member added to the structure by the type itself
    implicit getLength() {
    	return 1 + (this.next?.getLength() ?? 0)
	}
}
const obj: LinkedList = {
    value: 10,
    next: null
}
console.log(obj.length);
```



While meta types do act as templates, that’s not the whole story. They also have something called *implicit structure.* Implicit structure is structure which is derived from the variable members of a meta type.

This correspondence works similarly to a typed rest parameter in a function, except that it uses key-value semantics and not sequential semantics. 

**A generic type can define a rest type parameter with a meta type annotation describing its structure.**

```typescript
meta type Foo := <
    Foo extends unknown
	ArrayFoo := Foo[]
>
declare function something
```

## Operators and Operations

There are a few operations we can define on meta types and meta objects. 

### Meta objects

Meta objects are value-like constructions in the world of types, and as such support value-like operators.

#### Instantiation

This version of the operator allows us to instantiate generic members by matching up types to type parameters by key. 

```typescript
// We've talked about this earlier
meta object Blah := <
    A = number,
    B =  string
>

declare function example<A, B>(): void

example<...Blah>()
```

#### Combination

This version of the operator is different, and is used to combine meta objects like the spread operator for objects.

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

However, it differs from the spread operator for objects in several ways.

1. It’s recursive, merging nested meta objects.
2. The objects must not have conflicting members.

### Meta types

Meta types are types, and as such support a number of operators that work on types.

#### Indexing

This is the only operator I see as essential for the proposal. It’s similar to the indexing operation on data types:

```typescript
type Foo = {
	bar: string
}

```





It works on simple meta types (i.e. not the results of something like a disjunction). The result of this operation is the constraining type of the given member. 

```typescript
meta type Meta := < Type extends string >
    
let a: Meta["Type"]
```



#### Disjunction

Disjunction can be used between two meta types, but not between a data type and a meta type. So for example something like this is not allowed:

```typescript
// BAD!
string | < A extends string >
// BAD!
```

This, however, is allowed:

```typescript
meta type Foo := 
    | < A extends string, Type := "OneThing" > 
    | < B extends string, C extends string, Type := "Another" >
```

This works like union types work today – it must be disambiguated. The only way to do that is via a conditional involving a shared member of the meta type. So like this:

```typescript
type GetSomething<F: Foo> = F.Type extends "OneThing" ? F.A : F.B
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



However, if the two have contradictory constraints, this results in `never`. For example:

```
< A := string > & <
```



However, only some constraints can be merged. 

However, some constraints can’t be merged.

A union type must be disambiguated using a conditional for it to be useful. Since I don’t want to support putting meta types in conditionals directly, 

```typescript
Foo extends < A extends string> ? Foo.A : Foo.B
```





 `string | < A extends string >` is not allowed

```typescript

```



Disjunction creates a union meta type that works like a union type. Specifically, 





## Meta type



I decided to add this part here, because otherwise meta object inference and checking might not make sense. Feel free to skip it if you’re fine with takin

Consider a meta type such as the following:







## Referencing 

There are a few other things.

### 



These constraints are allowed to reference other meta members or the member being constrained, in the same way type parameters might do so:I said earlier that **a meta type is the type of a meta object**. 

The **meta object ⇔ meta type** relationship is the same as the **object ⇔ object type** relationship. **That is, meta objects are the instances of meta types.** Meta types can also be regarded as a structural template that some meta objects match and others don’t.



1. Meta annotations, used for meta object members.
2. Type constraints, used for type members.

This is because they are one level above the normal type system

Like everything else, meta types are structural. The core structure of a meta type describes the meta objects which are its instances. This is analogous to how object types describe object instances. 

Because meta types are one level above the normal type system, they can use two different kinds of constraints. 

Object types describe object values using type annotations, but meta types can have several kinds of members.

1. **Meta object members**, also called **meta members**, which use **meta annotations**.
2. **Type members**, which use **type constraints**.



**The constraint of a member is part of its declaration**, and it’s required. This constraint must specify, implicitly or explicitly, whether the member is a meta object or a type.

Specifically, meta types support the following constraints:

1. **Subtype constraints**, e.g. `Member extends string`

2. **Meta annotations**, e.g. `Member: MetaType`

3. **Equivalence constraints**, e.g. `Member := number`

4. However, **meta types possess another form of structure altogether.**  This structure is called **implicit structure**, and its existence is only possible because the instances of meta types are still part of the type system.

   Instead of being constraints on the structure of a meta object, implicit structure expands a meta object conforming to a meta type with additional structure, like additional types, derived from the structure it already has.

   As an analogy, consider Scala’s trait system, which can extend a type with additional members. The main difference is that **the mechanism is one level above Scala’s trait system.** Where Scala’s traits extend classes with extra functionality, implicit structure extends meta objects with extra type definitions.

   While it may sound weirdly abstract, actually implicit structure addresses multiple feature requests and without it, inference would probably be impossible in many cases.

   This means that working with meta types can, potentially, be an additive process. 
