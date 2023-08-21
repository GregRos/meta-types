In this case, the caller will still be able to construct the type manually like any other constrained variable. 

Using the `:=` constraint, the only possible inference 

Let’s take a look at the instances of that meta type.

```typescript
< Input := string, Predicate := (x: string) => string >
< Input := number, Predicate := (x: number) => number >
< Input := "abc",  Predicate := (x: "abc") => "abc" >
```

That’s incredibly uniform. So uniform, in fact, that you could imagine not needing to specify `Func` at all and have inference figure it out. And this is exactly what happens during meta object inference, which allows you to specify only some of the variables.

In fact, equivalence constraints are strictly constructive in a sense – 

Equivalence constraints only work like that if they don’t reference any variables, and this is exactly where the two concepts diverge. 

```typescript
<
	Input extends unknown
	Predicate := (x: Input) => boolean
>
```

In spite of that,   

```typescript
< Input := string, Predicate := (x: string) => string >
< Input := number, Predicate := (x: number) => number >
< Input := "abc",  Predicate := (x: "abc") => "abc" >
```





```typescript
< Stack := { next: Stack | null } >
```

Or you could even write:

```typescript
< Liar := Liar extends true ? false : true >
```

This shows that the equivalence constraint is, in fact, still a constraint. Special cases like `< x := string >` aside, it’s non-constructive and can still result in a contradiction. 

From this example, we can see that the equivalence example is a constraint.

Here we can see that the equivalence constraint is truly a constraint, rather than some kind of constant. `Example.Func` corresponds to different types in different instances of `Example`. Look at these, for example:

```typescript
< Input := string, Predicate := (x: string) => string >
< Input := number, Predicate := (x: number) => number >
< Input  := "abc",  Predicate := (x: "abc") => "abc" >
```

That’s incredibly uniform. So uniform, in fact, that you could imagine not needing to specify `Func` at all and have inference figure it out. 

We’ll come back to that later, but remember that the equivalence constraint is still a **constraint**. There doesn’t have to be a type matching that constraint. It’s even possible to use this fact in order to constrain the whole meta type indirectly.

```
type Invariant<T> = T extends { hello: string } ? true : false
```





Here are some non-instances:

```typescript
< Blah := string >
// Doesn't have member 'Input'

< Input := string, Func := number >
// number is not equivalent to (x: string) => string
    
< Input := < > >
// Expected Input to be a data type.

< Input 
```



It’s a **constraint**, not necessarily a constructive statement. 

You can use it to phrase tautologies, in principle. There is one restriction – since this kind of constraint can apply to both meta object members and data types members, your equivalence constraint 

```typescript
< A := A >
```

spot

It’s essentially the equivalent of the type `{a: “x”}` and the value `{a: “x”}`. We allow both to use the same notation because of a relationship between them, and the compiler never gets confused because they occur at different positions.



Anyway, phrasing silly meta types is far from this constraint’s only use case. It allows you to define type 









I’ll talk about the algorithms for these things 

While it’s clear to us this meta type can have no instances, the compiler probably won’t be able to tell that. The general problem of type equivalence is probably undecidable for complex types like these. It’s not even 



For example, the type `{a: number}` has lots of different instances, like `{a: 1}` and `{a: 0.5}`. 

Consider `{a: number}`. This type has lots of runtime instances, such as `{a: 1}`, `{a: 0.5}`, and so on. It acts as a kind of template, 



This is quite literally true. Equally, meta objects are instances of meta types.

A meta type is the type of a meta object – quite literally. 

Object types 

Once again, meta types are described using an expression that can be bound to name via a declaration.

Here is an example of a simple meta type, along with one of its instances.

```typescript
meta type Metatype = <
    // AnyType must be a data type
	AnyType extends unknown;
	// AnyString must be a subtype of string
	AnyString extends string
	// Subtype must be a subtype of AnyString, another type variable
	Subtype extends AnyString
>
        
//					↓ meta type annotation makes sure it's an instance
meta object Instance: Metatype = <
    AnyType := number,
    AnyString := "a" | "b",
    Subtype := "a"
>
```

So let’s unpack the important parts. First, notice that every member variable in `Metatype` must have a constriant. Metatypes wouldn’t very useful without these constraints. These constraint serve the same role as type annotations in a regular object type.

However, because we’re one level up, we can have two sorts of members in our meta type: type variables and meta object variables. Meta object variables describe other meta objects. So meta types can be nested just like object types.

```typescript
meta type HasTypeObjectMember = <
	//      ↓ you don't have to declare meta types
	Nested: < 
		Inner extends unknown 
	>
>

meta object Instance: HasTypeObjectMember = <
	Nested := <
		Inner := boolean
	>
>
```

But that’s not all! We can also have an equivalence constraint. This works for meta object variables as well as for type variables.

```typescript
meta type HasEqConstraints = <
	InputType extends unknown
	ArrayOfIt := InputType[]
	FuncOfIt := (x: InputType) => InputType
	TypeObjOfIt := <
        Whatever := InputType
	>
>

meta object Instance: HasEqConstraints = <
	InputType := string,
	ArrayOfIt := string[],
	FuncOfIt := (x: string) => string,
	TypeObjOfIt := <
        Whatever := string
	>
>
```

You might notice something weird – even though I used `:=` in the meta type, I still had to fill in all those members. Isn’t that weird?

Actually, that’s a tricky question! I didn’t have to do that and also it’s not weird. Members with `:=` constraints in a meta type are still constrained type variables, just like ones constrained with  `extends`. Because the constraints can use 

Well, that leads us to a crucial point of meta type science –  

f

**Every member of a meta type is a constrained type variable.** In fact, the constraint is not optional and determines the kind of member you’re defining. 

1. An `extends X` constraint places a subtype constraint, and also marks the member as being a data type rather than a meta object.
2. 

You see, some of these type variables can be occupied by meta objects instead. 

Several 

Several kinds of members are possible:





While the structure of meta objects is self-explanatory, the members of meta types require some explanation.

Every m

Meta types are like generic signatures encapsulated in individual objects. Just by replicating the capabilities of a generic type, they can 

### The type bag pattern

This pattern is examplified in [this issue](https://github.com/microsoft/TypeScript/issues/54254). Basically, instead of using multiple type parameters, some library authors instead encapsulate them in an object, and use a subtype constraint.

```typescript
interface TypeBag {
	readonly ParamOne: unknown;
	readonly ParamTwo: unknown;
}


```



> A small problem – without the HKT extension, meta objects can’t contain generic types. That means that the full integration of namespaces 

The power of the meta type system comes from abstracting over meta objects, but they have some use-cases just by themselves.

* Meta objects 

This meta object can be used as-is like a namespace that can only contain types. For example, you could write `let x: TypeObject.B`. Another use would be generic instantiation. 

* Meta objects use the same structure as object types `{…}` to represent a different concept.
* Meta objects support an equivalence relation, as well as something called a substructure relation.
* Meta objects support a kind of spread operator, as well as an intersection operator. 
* They can be naturally extended to support embedded generic types in the future, giving us HKTs.

## Existing constructs through meta types

One reason I call meta types a “natural extension” is how it corresponds to existing patterns, 



In general, there are several kinds of new problems the compiler needs to solve that concern meta types.

1. Type checking – given a meta object, determine if it’s an instance of a meta type.
2. Meta object inference – construct a meta object which is an instance of a meta type, given a certain typing environment. We’ll talk about that later.
3. Subtype relation – Determine if a meta type is a subtype of another meta type. This is needed in certain deeply generic constructions.





The meta type system is constructed similarly to HKTs in Haskell but results in different characteristics. Haskell spot

The core building block of this system is something called a **meta object**.



 **A type-level structure that embeds types like normal objects embed values**. It can be used to produce proper types by navigating object structure in the same way runtime code navigates objects to produce values.

Meta objects are abstracted over using **meta types**, higher-order types that are defined using some of the same language used to define generic signatures. Meta types naturally support things such as a subtype relation, union and intersection operators, and so on.

I discovered meta objects while attempting to introduce HKTs (Higher-Kinded Types) to TypeScript, but I quickly realized that the resulting structures could be used for more than that, and that the HKTs I got were actually an extension of a more basic concept. 

The significance of the meta type system can be seen in the large number of feature requests it addresses as well as the multiple connections it has to existing type system constructs. 

One way to look at the meta type system comes from a pattern often used by library authors. This pattern has been called a “generic bag” or a “type bag” and uses object types. The pattern is meant to encapsulate sets of type parameters as a single unit.

While the “type bag” pattern works, it’s more naturally expressed using meta objects and meta types than the way it’s expressed right now.

```typescript
interface GenericBag {
    OneType: unknown
    OtherType: unknown
}

interface HasGenericBag<Bag extends GenericBag> {
    getOne(): Bag["OneType"]
    getOther(): Bag["OtherType"]
}

const example = new HasGenericBag<{
    OneType: number
    OtherType: string
}>();
```

In the example above, the type parameter **Bag** can be regarded not as the type of a runtime object of the form `{OneType: 1, OtherType: “hello”}`, but rather as a **meta type parameter** that expects a meta object containing two types. 

Another example of a meta object can be seen in the type declaration component of a namespace. 

```typescript
namespace JustTypes {
	type A = {
		value: number
	}
	namespace Nested {
		type B = string
	}
}
```

The type structure of `JustTypes` doesn’t exist in the value binding `JustTypes` nor the type referred to as `typeof JustTypes`. At present it forms a kind of wrinkle in the type landscape. However, under this system, it becomes a third entity – a meta object also named `JustTypes`.

Yet another example is in the following correspondence:

1. Generic signature ⇒ Meta type
2. Generic instantiation ⇒ Meta object

This correspondence can be used to define an object-like rest/spread syntax that mirrors rest parameters in some ways, though in this case the parameter is object-like rather than sequential. 

```typescript
declare function generic<A, B>(x: A): void;
generic<...TypeObject>(null!)
generic<...<A = number, B = string>>(null!)

declare function restParam<...Rest: MetaType>(x: A): void;
restParam<number, string>(null!)
```

## Meta objects

* 

## The meta type

Meta types can be given as an expression or declared. A meta type declaration looks like this:

```typescript
meta type MetaType = <
    A extends unknown
    B extends string
    C: OtherMetaType
    D = A[]
>
```

Meta types can be used to **encapsulate generic signatures into a logical unit,** which users can work with using the same language as regular types.

* They have instances (meta objects).
* They have a well-defined subtype relation.
* They can support union and intersection operators.
* They can built on other meta types through composition.
* They can express families of closely related types.
* They can be extended to support embedded generic types in the future.

## Meta type annotations

Both meta types and meta objects are type system constructs that undergo type erasure and don’t affect runtime code. They can be primarily used as type parameters.

A type parameter can be transformed into a **meta object parameter** by using a **meta type annotation** which looks just like a regular type annotation:

```typescript
meta type MetaType = <
    A extends unknown
>

declare function example<TO: MetaType>(x: TO.A): TO.A
```

To call `example` we need to supply two things:

1. A meta object that is an instance of `MetaType`, such as `<A = number>`. 
2. A value that conforms to `TO.A`.

In principle, inference can construct a meta object   



This isn’t cosmetic - meta types naturally support type operators such as union and intersection. They have a well-defined subtype relation. They can be nested to form complex relationships between types and they can describe recursive constraints on types using natural self-references.



Carefully designing a meta type-based API can create 

Regular objects can be abstracted over using object types; meta objects can be abstracted over using a system I call **meta types**. A meta type is a higher-order type that applies TypeScript’s structural type system to meta objects, resulting 





Meta objects aren’t very useful by themselves. 

 It can be used to produce regular types by navigating its object structure. 

For meta objects to be more useful than a namespace 

In order to meaningfully use meta objects

{! DRAWBACKS OF GENERIC BAGS !}



This pattern

I'd like to explain that statement in comparison to another language - Haskell. Haskell is a purely functional language, and it’s exceptionally good at reasoning about and working with functions. It has many functional operators such as the pipe operator, the composition operator, and so on. Because in Haskell everything is a function, more or less, it's natural to apply the language of functions to the world of types. This results in **type constructors** - functions taking types as inputs and returning types as outputs.

While type constructors are defined to be types, they behave more like function values in the world of types, type-level functions. You can call one with an input type, giving you an output type, but you can’t use them to annotate variables or anything like that. These higher-order values have matching higher-order types known as *kinds*. Anything that's not a type constructor has the structureless *kind* of `*`, while type constructors have *kinds* that describe their function structure, such as `* -> *`. This is why type constructors are confusingly called HKTs, Higher-Kinded Types.

**TypeScript, however, is not Haskell.** For one thing, it's not nearly as good at talking about functions. It is, however, amazing at talking about *object structure*, and it gives developers powerful tools for working with it. While TypeScript supports functions, they are essentially objects with call signatures. 

If we follow Haskell’s lead by porting a pattern from the runtime world to the type world, what we’d get isn’t type-level functions but rather type-level objects. These are higher-order objects that embed types in the same way regular objects embed values like 5. 

Then, in accordance with Haskell’s pattern, we’re going to define a **higher-order type** called a *kind* that allows us to abstract over many different type-level objects.

> ### Type-level objects versus object types
>
> You might wonder how that is different from an object type. After all, we also give it as types embedded in an object structure:
>
> ``` typescript
> // Not a meta object!
> {
> 	a: string,
> 	b: number
> } 
> ```
>
> **Object types and meta objects represent different concepts with the same information**. They differ in the operations each supports and the context in which it’s used. The best comparison is to the relationship between 2d vectors and complex numbers.

You might be wondering what meta objects are good for. The answer is a lot of things! Meta objects answer lots of existing use-cases and their implementation will result in an evolution of TypeScript’s powerful type system. We’ll take a look at all the possible use-cases in this proposal, but because they are a tricky concept to grasp I’d like to first look at their behavior and characteristics.

## Declaring meta objects

Meta objects can be given as expressions or declared like other components of the type system. Here is how you might declare one:

```typescript
meta object Example = <
    A = number
    B = string
>
```

Note the usage of `<…>`. This syntax is meant to emphasize that meta objects are closely related to **type parameters**.

By themselves, meta objects are nothing special. In fact, they already exist in the language! The type declarations of a namespace form a meta object. **The easiest way to understand meta objects is as namespaces that can only contain types.**



You might wonder how that is different from an object type. After all, we also give it as types embedded in an object structure:

``` 
{
	a: string,
	b: number
} 
```

**Object types and meta objects represent different concepts with the same information**. It’s kind of like how a 2d point is a 2d vector which is also a complex number. They can also be extended differently - object types can be extended through union and intersection, while meta objects can be extended to support embedded generic types. But you could use one to represent the other in a pinch.

Type-level objects are an incredibly useful feature that addresses existing use-cases, simplifies complex type definitions, and can be extended 

## That sounds familiar

You might wonder how that is different from an object type. After all, we also give it as types embedded in an object structure:

``` 
{
	a: string,
	b: number
} 
```

**Object types and meta objects represent different concepts with the same information**. It’s kind of like how a 2d point is a 2d vector which is also a complex number. They can also be extended differently - object types can be extended through union and intersection, while meta objects can be extended to support embedded generic types. But you could use one to represent the other in a pinch.

## They’re already here

**Actually, meta objects already exist!** We just don’t have the tools to deal with them.

They exist in two ways:

1. As a pattern that’s used by many library authors.
2. Namespaces.

### The meta object pattern

A common pattern used by many library authors is the following:

```typescript
export interface TypeObject {
    readonly Type1: string;
    readonly Type2: number;
}
export interface Example<TO extends TypeObject> {
    hello(): TO["Type1"]
    goodbye(): TO["Type2"]
}
```

This pattern represents meta objects using object types and attempts to quantify over them using tools for quantifying over types (the `extends` constraint). This sometimes works, but it’s not very user friendly due to issues ranging from inference to amibiguity. It’s also hampered by how the language relaxes the assignability relation in unsound ways.

As a general rule, the tools for dealing with object types aren’t very good for dealing with meta objects. New tools are needed to properly implement this pattern.

### Namespaces

Let’s say we have the following code:

```typescript
namespace Namespace {
	export const id = <T>(x: T) => x;
	export type Blah = {
		value: number
	}
}
```

This code declares three different entities under the `Namespace` moniker:

1. The value binding `Namespace`.
2. The type referred to using `typeof Namespace`.
3. A third entity, a meta object also called `Namespace`, which has the member `Namespace.Blah`. 

## The definition

Okay, enough with the premable. Let’s declare some meta objects!

```typescript
meta object Example = <
    Type1 = number
    Type2 = string
    Nested = <
    	meta type3 = boolean
	>
>
```

This syntax declares a meta object called Example. **This meta object behaves just like a namespace that can only contain types.** You can use it to produce different types, such as `let a: Example.Type1` where we use it to produce the type `number`.

Being values in the land of types, meta objects 

```
meta type Thing = <
	Type1 extends number
	Type2 extends string
>

meta type ThingExtended = Thing & <
	Type3 extends string
>
```





The type declarations of a namespace **are meta objects**. 



I think it’s a good idea to have different notation

I’m going to define a new syntax for meta objects, but the syntax isn’t necessarily required since we already have one. I think it’s important because 

I do think it’s important b ecause meta objects and object types are very different entities. 



Well, the main difference is how function types work. They are an extension, though, and if we ignore them there isn’t any difference. It’s like how a point is a 2d vector which is also a complex number. All of these things contain the same information and the only difference is the operations defined on them.

The operations are still important, though, as is the ability to generalize. A function type maps to a type-level function. 

> **How it works**
> Types exist to abstract over runtime values. A type is essentially a set of runtime values which are its instances. `never` is the empty type, corresponding to the empty set ∅. The type `number` is the set of all number values. 
>
> An object type is a type that abstracts over lots of runtime object values that share some of their object structure. While object types also have
>
> This is very different from an object value 

<img src="./assets/image-20230816221136120.png" alt="image-20230816221136120" style="zoom:50%;" />

This isn’t a feature that exists in other languages. 

This is why if we follow Haskell’s lead we get not type-level functions but type-level objects. 

We can’t do exactly what Haskell does, because it’s a very different language, but we can follow the same pattern. Instead of functions, we apply the structure of an object to the world of types. What we get aren’t object types - they aren’t types at all - but rather higher-order objects that embed types in the same way regular objects embed values.





It has constructs that map one object structure to another, operators that merge object structures, and so on. Functions aren't first-class in TypeScript - you can work with them very well, but they're still object types with call signatures.



This is why trying to port Haskell's type constructor features to TypeScript directly has generally failed. HKTs, or at least the way Haskell defines them, are not native to 

`*`, which means that normal values lack higher-order structure. Meanwhile, the *kind* of a type constructor is something like `* -> * -> *`. That is, it's like a template that describes a range of type constructors of which individual type constructors are instances.

hey don't have instances. Instead, they are higher-order function values that exist in the world of types. 

TypeScript isn't as good at working with functions. Instead it's really good at working with object structure. It has special operators for merging objects, mapping one structure to another, and so on. 

All of this means that type constructors aren't as natural to TypeScript as they are to Haskell. But that doesn't mean we can't make use of the same pattern and see where it gets us. Namely, **taking a pattern central to the language - in our case, the object - and applying it to the world of types**.

## But what is it?

A meta object is an object full of types.

Runtime object structure can be regarded as a labeled tree with runtime values, such as `4` and `"hello world"` at its leaves. A meta object also has object structure, but instead of having runtime values for leaves it has types. These types are actually embedded in the structure, rather than being annotations that describe a range of possible values.

Here is an example of a meta object:

```typescript
meta object Example = {
	type A = {
    	value number
	}	
}
```

A **meta object is not a type itself**. It's an application of object structure to the world of types, where types are seen as values. This is similar to how type constructors aren't types in the normal sense - they have no instances - but rather type-level functions.

**Actually, meta objects already exist in the language.** They are what happens when you define a namespace that contains types.

As such, just talking about meta objects doesn't give us anything new. However, just like Haskell allows type parameters that must be type constructors, TypeScript can accept meta objects as type parameters. 

[ NAMESPACES ]

f

TypeScript is not as good at working with functions. Instead, it's really good at working with object structure and its ability to reason about functions is an extension of that. This is also true for JavaScript. JavaScript doesn't have operators that compose functions - it does, however, have operators that compose objects.

I think it makes sense, then, that if TypeScript were to use its object-based type system to reason about its own types, the result would be a types embedded in an object structure. 

That doesn't really correspond to a feature that exists in other languages, but TypeScript is different from other languages so it would make sense for it to have type system features that don't exist anywhere else, just like Haskell has many features that are unique to it.



This is not a feature that exists in other programming languages, but TypeScript is different from other languages at its core. No other major language shares its structural type system. 

As an extension of this, Haskell supports what are termed HKTs or type constructors - functions that accept and return types. 

also supports what are termed Higher-Kinded Types (HKTs). HKTs in Haskell are type constructors and operators - functions that map types to other types, which can be used 

TypeScript is the best at reasoning about object structure. Its subtype relation is inherently tied to object structure, it has special syntax for producing complex object types, and so on. This is kind of similar to a language like Haskell, which is incredibly good at reasoning about functions.

When Haskell applies the type system to itself, the natural result is **type constructors** - functions that take types as arguments and return types. These type constructor functions can then be used to quantify over what we'd call generic types. This is the storied HKT (Higher-Kinded Type) feature that some people have wanted.

Hello! I have a weird question. How controversial is the following statement: Type constructors (in Haskell) don't behave like types. Instead they behave like higher-order function values that exist in the world of types.

1. 



actually one of the purposes of this constraint is *avoiding* repetition.







constraint involves a lot of repetition, and you’d be right. Luckily, there is a trick which I haven’t mentioned yet – **meta object inference**. This is when the user supplies a partial meta object, giving only some of its structure, and lets the compiler deduce the rest of the structure through inference.

For example, the following meta object is partial to the meta type we defined earlier:

```typescript
< X := number/*, Predicate := (x: number) => boolean */ >
```

The right-hand side of the equivalence constraint also gives instructions on how to construct the type `Predicate`.

Using the `:=` constraint guarantees that inference will construct equivalent types, given the same typing environment, forcing it to be a fairly deterministic process. This means that equivalence constraints on meta types can be used to define aliases and associated types, allowing users to avoid repetition.

The equivalence constraint only acts like a type alias, though. 