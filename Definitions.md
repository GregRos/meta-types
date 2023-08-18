# Definitions

## Type Objects

Type objects, like most type system constructs, can be declared or given as an expression. A type object expression has the form:

```typescript
// Other notations are possible. I call this the < > notation.
< 
    A := number, 
    B := <
    	C := string
	>
>
```

With the empty type object being represented as `< >`.

This is an object structure surrounded by `<…>`, where every leaf is a type. It’s pretty much identical to an object type, but the syntax is different because it represents a different  concept.

A type object declaration associates this expression with a name. This can be exported, imported, and so on – like all other declarations.

```typescript
type object TypeObject = <
    A := number,
    B := string,
    C = <
    	D := object, E := number
    >
>
```

To better understand type objects, let’s compare them to existing structures and patterns in TypeScript code.

### Namespaces

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

The third entity corresponds to a type object also named `Example`. 

**In other words, type objects are like namespaces that can only declare types, which are all exported.**

This means that type objects can be used just like namespaces, even though this isn’t the main way you’d use them.

```typescript
let a: Example.A
```

### Generic type instantiations

One of the reasons behind the `<…>` notation is the correspondence between type objects and the instantiations of generic types.

Let’s say you have a generic type, and we instantiate it with a set of type parameters.

```typescript
type Example<A, B> = [A, B]
type Instantiated = Example<number, string>
```

Then the instantiation correspondence to a type object of the form:

```typescript
<
	A := number,
	B := string
>
```

In fact, this allows us to define a **natural spread operator for type parameters**. This operator works using key-value semantics rather than sequential semantics.

```typescript
type object Types = <
    A := number,
    B := string
>
    
type Instantiated = Example<...Types>
```

This is one way in which Meta Types simplify and restructure complex generic types – by turning complicated instantiations into declared entities with their own structure.

## Meta types

Type objects are the workhorse of Meta Types, but the meta type get all the credit because they’re just so cool.

I said earlier that **a meta type is the type of a type object**. It’s like a template that some type objects match and others don’t. The template uses a combination of structure and several different types of constraints. It’s like a higher-order object type, but being higher-order lets it be much more expressive.

Meta types can declare one of the following constraints:

1. Subtype constraints
2. Meta type annotations
3. Equivalence constraints

Let’s use the subtype constraint to investigate how meta types work in general.

### Subtype constraints

This constraints a variable to be a subtype of a data type. Any member constrained using a subtype constraint is automatically regarded as a data type.

```typescript
type type Simple = <
	Sth extends unknown
>
// Similar to type object notation, but type objects must specify all members.
// Type objects and meta types can't appear in the same contexts so it's never ambiguous.
```

Once again, meta types are written as expressions. Declaring them simply binds them to a name. Like other declarations, meta type declarations can be exported and imported.

Here are some instances of said meta type. As with everything else in TypeScript, determining if a type object is an instance of a meta type is done structurally, where structure is defined very similarly to object type structure. 

```typescript
// Sth can be any data type
< Sth := 1 | 2 >
< Sth := number >
< Sth := string >
< Sth := { name: string, id: number } >

// Type objects can define more structure if they want.
< Sth := number, Else := string >
< Sth := number, Else := < Boo := string > >
```

On the other hand, the following aren’t instances of it:

```typescript
// ! COUNTER-EXAMPLES !
< Boo := number > 
// Structure doesn't match!
    
< Sth := < A := 1 > > 
// Expected Sth to be data type, but it was a type object!
    
< Sth extends string > 
// Nice try, but it's just another meta type. Type objects always specify all members! 
    
< >
// It's empty
    
< Sth = 10 > 
// Syntax error    
    
{ Sth: string }
// What? This is a data type.
// ! COUNTER-EXAMPLES !
```

Every member specified by a meta type must be constrained somehow. There are several possible constraints, and these constraints can reference ambient types or give types as inline expressions. For example:

```typescript
< Sth extends { complicated: Record<string, {a: 1}> } >
```

**Member constraints can also reference other variables in the meta type or themselves,** which allows them to represent complex webs of interacting types. For instance:

```typescript
<
	Tree extends {
		left: Tree | null
		right: Tree | null
	}
	Visitor extends {
		visit(tree: Tree);
	}
>
```

The same ability also lets us specify circular constraints.

```typescript
< Stack extends { readonly tail: Stack | null } >
```

There are two edge cases when you allow stuff like this. The first one is the tautology:

```typescript
< Sth extends Sth >
```

This meta type looks strange, but it’s actually equivalent to `< Sth extends unknown >`. The constraint `Sth extends Sth` is a tautology that’s true for any data type.

We can do the opposite true:

```typescript
< Liar extends (Liar extends true ? false : true) >
```

That’s a contradiction. It doesn’t break the language or something – it just means that this type has no instances, and thus is equivalent to `never`.

These examples are odd, but type checking is already [undecidable](https://github.com/microsoft/TypeScript/issues/14833) due to other factors. Being able to phrase contradictions and strange self-references is the cost of a powerful type system.

### Meta type annotations

A meta type annotation is written in the same way as a type annotation, and denotes that a member variable must be an instance of a meta type. This kind of constraint identifies a variable as being a type object and not a data type.

```typescript
type type Example = <
    X extends unknown
>

< Sth: Example >
```

Here are some instances:

```typescript
< Sth := < X := string > >
< Sth := < X := number > >
< Sth := < X := never > >

// Once again, type objects can also have additional structure.
< Sth := < X := string, Y := number>, Else := string >
```

Here are some non-instances:

```typescript
// ! COUNTER-EXAMPLES !

< Sth := 5 >
// Expected Sth to be a type object, but it was a data type.
    
< Sth := < Y := string > >
// Sth doesn't have the member X.

< Sth: Example >
// This is another meta type, not a type object.
    
// ! COUNTER-EXAMPLES !
```

### Equivalence constraint

The equivalence constraint is, in some ways, the trickiest one. It’s also extremely useful. It works for both data type and type object variables, 

```typescript
// This is both a type object and a meta type
// Just like {a: "x"} is both a value and a type.
< A := string >
```

It uses exactly the same syntax as type objects, and this is deliberate. 

This is the meta type equivalent of the relationship between the type `5` and the value `5`. They will always occur at different positions, so the compiler will never be confused about which one we mean, and there is a close correspondence between them.

Equivalence constraints only work like that if they don’t reference any variables, and this is exactly where the two concepts diverge. Here is an example of something pretty cool:

```typescript
type type Example = < 
    Input extends unknown
    Func := (x: Input) => Input
>
```

Here we can see that the equivalence constraint is truly a constraint, rather than some kind of constant. `Example.Func` corresponds to different types in different instances of `Example`. Look at these, for example:

```typescript
< Input := string, Func := (x: string) => string >
< Input := number, Func := (x: number) => number >
< Input := "abc",  Func := (x: "abc") => "abc" >
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

You can use it to phrase tautologies, in principle. There is one restriction – since this kind of constraint can apply to both type object members and data types members, your equivalence constraint 

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



This is quite literally true. Equally, type objects are instances of meta types.

A meta type is the type of a type object – quite literally. 

Object types 

Once again, meta types are described using an expression that can be bound to name via a declaration.

Here is an example of a simple meta type, along with one of its instances.

```typescript
type type Metatype = <
    // AnyType must be a data type
	AnyType extends unknown;
	// AnyString must be a subtype of string
	AnyString extends string
	// Subtype must be a subtype of AnyString, another type variable
	Subtype extends AnyString
>
        
//					↓ meta type annotation makes sure it's an instance
type object Instance: Metatype = <
    AnyType := number,
    AnyString := "a" | "b",
    Subtype := "a"
>
```

So let’s unpack the important parts. First, notice that every member variable in `Metatype` must have a constriant. Metatypes wouldn’t very useful without these constraints. These constraint serve the same role as type annotations in a regular object type.

However, because we’re one level up, we can have two sorts of members in our meta type: type variables and type object variables. Type object variables describe other type objects. So meta types can be nested just like object types.

```typescript
type type HasTypeObjectMember = <
	//      ↓ you don't have to declare meta types
	Nested: < 
		Inner extends unknown 
	>
>

type object Instance: HasTypeObjectMember = <
	Nested := <
		Inner := boolean
	>
>
```

But that’s not all! We can also have an equivalence constraint. This works for type object variables as well as for type variables.

```typescript
type type HasEqConstraints = <
	InputType extends unknown
	ArrayOfIt := InputType[]
	FuncOfIt := (x: InputType) => InputType
	TypeObjOfIt := <
        Whatever := InputType
	>
>

type object Instance: HasEqConstraints = <
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

1. An `extends X` constraint places a subtype constraint, and also marks the member as being a data type rather than a type object.
2. 

You see, some of these type variables can be occupied by type objects instead. 

Several 

Several kinds of members are possible:





While the structure of type objects is self-explanatory, the members of meta types require some explanation.

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



> A small problem – without the HKT extension, type objects can’t contain generic types. That means that the full integration of namespaces 

The power of the meta type system comes from abstracting over type objects, but they have some use-cases just by themselves.

* Type objects 

This type object can be used as-is like a namespace that can only contain types. For example, you could write `let x: TypeObject.B`. Another use would be generic instantiation. 

* Type objects use the same structure as object types `{…}` to represent a different concept.
* Type objects support an equivalence relation, as well as something called a substructure relation.
* Type objects support a kind of spread operator, as well as an intersection operator. 
* They can be naturally extended to support embedded generic types in the future, giving us HKTs.

## Existing constructs through meta types

One reason I call meta types a “natural extension” is how it corresponds to existing patterns, 



In general, there are several kinds of new problems the compiler needs to solve that concern meta types.

1. Type checking – given a type object, determine if it’s an instance of a meta type.
2. Type object inference – construct a type object which is an instance of a meta type, given a certain typing environment. We’ll talk about that later.
3. Subtype relation – Determine if a meta type is a subtype of another meta type. This is needed in certain deeply generic constructions.





The meta type system is constructed similarly to HKTs in Haskell but results in different characteristics. Haskell spot

The core building block of this system is something called a **type object**.



 **A type-level structure that embeds types like normal objects embed values**. It can be used to produce proper types by navigating object structure in the same way runtime code navigates objects to produce values.

Type objects are abstracted over using **meta types**, higher-order types that are defined using some of the same language used to define generic signatures. Meta types naturally support things such as a subtype relation, union and intersection operators, and so on.

I discovered type objects while attempting to introduce HKTs (Higher-Kinded Types) to TypeScript, but I quickly realized that the resulting structures could be used for more than that, and that the HKTs I got were actually an extension of a more basic concept. 

The significance of the meta type system can be seen in the large number of feature requests it addresses as well as the multiple connections it has to existing type system constructs. 

One way to look at the meta type system comes from a pattern often used by library authors. This pattern has been called a “generic bag” or a “type bag” and uses object types. The pattern is meant to encapsulate sets of type parameters as a single unit.

While the “type bag” pattern works, it’s more naturally expressed using type objects and meta types than the way it’s expressed right now.

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

In the example above, the type parameter **Bag** can be regarded not as the type of a runtime object of the form `{OneType: 1, OtherType: “hello”}`, but rather as a **meta type parameter** that expects a type object containing two types. 

Another example of a type object can be seen in the type declaration component of a namespace. 

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

The type structure of `JustTypes` doesn’t exist in the value binding `JustTypes` nor the type referred to as `typeof JustTypes`. At present it forms a kind of wrinkle in the type landscape. However, under this system, it becomes a third entity – a type object also named `JustTypes`.

Yet another example is in the following correspondence:

1. Generic signature ⇒ Meta type
2. Generic instantiation ⇒ Type object

This correspondence can be used to define an object-like rest/spread syntax that mirrors rest parameters in some ways, though in this case the parameter is object-like rather than sequential. 

```typescript
declare function generic<A, B>(x: A): void;
generic<...TypeObject>(null!)
generic<...<A = number, B = string>>(null!)

declare function restParam<...Rest: MetaType>(x: A): void;
restParam<number, string>(null!)
```

## Type objects

* 

## The meta type

Meta types can be given as an expression or declared. A meta type declaration looks like this:

```typescript
type type MetaType = <
    A extends unknown
    B extends string
    C: OtherMetaType
    D = A[]
>
```

Meta types can be used to **encapsulate generic signatures into a logical unit,** which users can work with using the same language as regular types.

* They have instances (type objects).
* They have a well-defined subtype relation.
* They can support union and intersection operators.
* They can built on other meta types through composition.
* They can express families of closely related types.
* They can be extended to support embedded generic types in the future.

## Meta type annotations

Both meta types and type objects are type system constructs that undergo type erasure and don’t affect runtime code. They can be primarily used as type parameters.

A type parameter can be transformed into a **type object parameter** by using a **meta type annotation** which looks just like a regular type annotation:

```typescript
type type MetaType = <
    A extends unknown
>

declare function example<TO: MetaType>(x: TO.A): TO.A
```

To call `example` we need to supply two things:

1. A type object that is an instance of `MetaType`, such as `<A = number>`. 
2. A value that conforms to `TO.A`.

In principle, inference can construct a type object   



This isn’t cosmetic - meta types naturally support type operators such as union and intersection. They have a well-defined subtype relation. They can be nested to form complex relationships between types and they can describe recursive constraints on types using natural self-references.



Carefully designing a meta type-based API can create 

Regular objects can be abstracted over using object types; type objects can be abstracted over using a system I call **meta types**. A meta type is a higher-order type that applies TypeScript’s structural type system to type objects, resulting 





Type objects aren’t very useful by themselves. 

 It can be used to produce regular types by navigating its object structure. 

For type objects to be more useful than a namespace 

In order to meaningfully use type objects

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
> // Not a type object!
> {
> 	a: string,
> 	b: number
> } 
> ```
>
> **Object types and type objects represent different concepts with the same information**. They differ in the operations each supports and the context in which it’s used. The best comparison is to the relationship between 2d vectors and complex numbers.

You might be wondering what type objects are good for. The answer is a lot of things! Type objects answer lots of existing use-cases and their implementation will result in an evolution of TypeScript’s powerful type system. We’ll take a look at all the possible use-cases in this proposal, but because they are a tricky concept to grasp I’d like to first look at their behavior and characteristics.

## Declaring type objects

Type objects can be given as expressions or declared like other components of the type system. Here is how you might declare one:

```typescript
type object Example = <
    A = number
    B = string
>
```

Note the usage of `<…>`. This syntax is meant to emphasize that type objects are closely related to **type parameters**.

By themselves, type objects are nothing special. In fact, they already exist in the language! The type declarations of a namespace form a type object. **The easiest way to understand type objects is as namespaces that can only contain types.**



You might wonder how that is different from an object type. After all, we also give it as types embedded in an object structure:

``` 
{
	a: string,
	b: number
} 
```

**Object types and type objects represent different concepts with the same information**. It’s kind of like how a 2d point is a 2d vector which is also a complex number. They can also be extended differently - object types can be extended through union and intersection, while type objects can be extended to support embedded generic types. But you could use one to represent the other in a pinch.

Type-level objects are an incredibly useful feature that addresses existing use-cases, simplifies complex type definitions, and can be extended 

## That sounds familiar

You might wonder how that is different from an object type. After all, we also give it as types embedded in an object structure:

``` 
{
	a: string,
	b: number
} 
```

**Object types and type objects represent different concepts with the same information**. It’s kind of like how a 2d point is a 2d vector which is also a complex number. They can also be extended differently - object types can be extended through union and intersection, while type objects can be extended to support embedded generic types. But you could use one to represent the other in a pinch.

## They’re already here

**Actually, type objects already exist!** We just don’t have the tools to deal with them.

They exist in two ways:

1. As a pattern that’s used by many library authors.
2. Namespaces.

### The type object pattern

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

This pattern represents type objects using object types and attempts to quantify over them using tools for quantifying over types (the `extends` constraint). This sometimes works, but it’s not very user friendly due to issues ranging from inference to amibiguity. It’s also hampered by how the language relaxes the assignability relation in unsound ways.

As a general rule, the tools for dealing with object types aren’t very good for dealing with type objects. New tools are needed to properly implement this pattern.

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
3. A third entity, a type object also called `Namespace`, which has the member `Namespace.Blah`. 

## The definition

Okay, enough with the premable. Let’s declare some type objects!

```typescript
type object Example = <
    Type1 = number
    Type2 = string
    Nested = <
    	type Type3 = boolean
	>
>
```

This syntax declares a type object called Example. **This type object behaves just like a namespace that can only contain types.** You can use it to produce different types, such as `let a: Example.Type1` where we use it to produce the type `number`.

Being values in the land of types, type objects 

```
type type Thing = <
	Type1 extends number
	Type2 extends string
>

type type ThingExtended = Thing & <
	Type3 extends string
>
```





The type declarations of a namespace **are type objects**. 



I think it’s a good idea to have different notation

I’m going to define a new syntax for type objects, but the syntax isn’t necessarily required since we already have one. I think it’s important because 

I do think it’s important b ecause type objects and object types are very different entities. 



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

A type object is an object full of types.

Runtime object structure can be regarded as a labeled tree with runtime values, such as `4` and `"hello world"` at its leaves. A type object also has object structure, but instead of having runtime values for leaves it has types. These types are actually embedded in the structure, rather than being annotations that describe a range of possible values.

Here is an example of a type object:

```typescript
type object Example = {
	type A = {
    	value number
	}	
}
```

A **type object is not a type itself**. It's an application of object structure to the world of types, where types are seen as values. This is similar to how type constructors aren't types in the normal sense - they have no instances - but rather type-level functions.

**Actually, type objects already exist in the language.** They are what happens when you define a namespace that contains types.

As such, just talking about type objects doesn't give us anything new. However, just like Haskell allows type parameters that must be type constructors, TypeScript can accept type objects as type parameters. 

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