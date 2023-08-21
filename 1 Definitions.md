# 1 Definitions

## Meta objects

Meta objects, like most type system constructs, can be declared or given as an expression. Here is a simple example:

```typescript
< 
    A := number, 
    B := string
>
```

**A meta object resembles an object value in the world of types.** Where normal objects embed values like `5` and `“hello world”`, meta objects embed values like `string` and `{ id: number, name: string }`.

Meta objects can be declared, exported, and imported. Here is how that looks

```
export meta object Something 
```



Like all things in TypeScript, meta objects are purely structural entities. 

To help is understand meta objects, let’s investigate the equivalence 

With the empty meta object being represented as `< >`.

This is an object structure surrounded by `<…>`, where every leaf is a type. It’s pretty much identical to an object type, but the syntax is different because it represents a different  concept.

A meta object declaration associates this expression with a name. This can be exported, imported, and so on – like all other declarations.

```typescript
meta object TypeObject = <
    A := number,
    B := string,
    C = <
    	D := object, E := number
    >
>
```

To better understand meta objects, let’s compare them to existing structures and patterns in TypeScript code.

### Correspondence 1: Generic type instantiations

One of the reasons behind the `<…>` notation is the correspondence between meta objects and the instantiations of generic types.

Let’s say you have a generic type, and we instantiate it with a set of type arguments.

```typescript
type Example<A, B> = [A, B]
type Instantiated = Example<number, string>
```

Then the instantiation correspondence to a meta object of the form:

```typescript
<
	A := number,
	B := string
>
```

In fact, this allows us to define a **natural spread operator for type parameters**. This operator works using key-value semantics rather than sequential semantics.

```typescript
meta object Types = <
    A := number,
    B := string
>
    
type Instantiated = Example<...Types>
```

This is one way in which Meta Types simplify and restructure complex generic types – by turning complicated instantiations into declared entities with their own structure.

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

**In other words, meta objects are like namespaces that can only declare types, which are all exported.**

This means that meta objects can be used just like namespaces, even though this isn’t the main way you’d use them.

```typescript
let a: Example.A
```

### Meta objects and references

Meta objects are value-like constructions in the world of types that describe a rigid and finite tree structure.

Because types aren’t part of that structure, the types in a meta object can reference other types in the same meta object, or even themselves. This works like types embedded in a namespace.

```typescript
namespace Illustration {
	type A = string
	type B = {
		value: A
	}
}
```

This means that the following are legal constructions:

```typescript
< A := string, B := {value: A} >
< A := { value: A } >
```

However, the structure of a meta object can’t be built using references like these. While non-cyclical reference can be allowed:

```typescript
< A := string, B := A >
```

Anything that resolves into a cycle isn’t. For example:

```typescript
// COUNTER EXAMPLES
< A := < X := A > >
< A := < X := B >, B := < X := A > >
// COUNTER EXAMPLES
```

## Meta types

Meta objects are the workhorse of Meta Types, but I’m giving meta types all the credit because they’re just so cool.

I said earlier that **a meta type is the type of a meta object**. It’s like a template that some meta objects match and others don’t. The template uses a combination of structure and several different types of constraints, resembling a higher-order object type.

While object types have value members, meta types have type and meta object members. These members can be of two different kinds:

1. Variable members
2. Computed members

While meta types are structurally typed, their computed members form **implicit structure**. This is structure that is implicitly added, or **mixed in**, into a conforming meta object during the process of conformation.

**Implicit structure is something object types just can’t have.** This is because the instance of an object type is a runtime object, which type information can’t affect. However, there is no problem with applying it to a meta object. 

However, because the instance of a meta type still belongs in the world of types, there is no problem with enriching it in this way. 

**However, because the instance of a meta type is a type-level structure, there is no issue with enriching it with additional type information in this way.** 



Two instances of the same meta type (that is, two meta objects) are equivalent if they have 

Meta types have two kinds of members.

1. Constrained member variables
2. c   

Constrained member variables correspond to constrained type parameters, while computed members are associated and derived types constructed from those parameters.

While both variables and 

When 

Whereas object types can have value members and method members, meta types have **meta members**, which can either be:

* Meta object members
* Type members

These members correspond to type parameters, and the meta type as a whole corresponds to a generic signature – but encapsulated in a single structural entity and enriched with additional constriants. These constraints are the source of a meta type’s expressive power.

**The constraint of a member is part of its declaration**, and it’s required. This constraint must specify, implicitly or explicitly, whether the member is a meta object or a type.

Specifically, meta types support the following constraints:

1. **Subtype constraints**, e.g. `Member extends string`
2. **Meta annotations**, e.g. `Member: MetaType`
3. **Equivalence constraints**, e.g. `Member := number`

These constraints are allowed to reference other meta members or the member being constrained, in the same way type parameters might do so:

```typescript
declare function foo<A, Stack extends {value: A, next: Stack | null}>(): void
```

Since all members must be constrained, let’s investigate meta types as a whole using the **subtype constraint**.

### Meta types via the subtype constraint

This constrains a variable to be a subtype of some other type, and works just like a subtype constraint in a generic signature.

Here is an example of a meta type that declares a single member using a subtype constraint:

```typescript
meta type Simple = <
	Sth extends unknown
>
// Similar to meta object notation, but meta objects must specify all members.
// Meta objects and meta types can't appear in the same contexts so it's never ambiguous.
```

Once again, meta types are written as expressions. Declaring them simply binds them to a name. Like other declarations, meta type declarations can be exported and imported.

The syntax of a meta type is similar to that of a meta object. There are several important differences, though:

1. Meta objects fully assign all their members to concrete types or other meta objects using `:=`.
2. Meta objects can’t reference other 

Here are some instances of said meta type. As with everything else in TypeScript, determining if a meta object is an instance of a meta type is done structurally, where structure is defined very similarly to object type structure. 

```typescript
// Sth can be any data type
< Sth := 1 | 2 >
< Sth := number >
< Sth := string >
< Sth := { name: string, id: number } >

// Meta objects can define more structure if they want.
< Sth := number, Else := string >
< Sth := number, Else := < Boo := string > >
```

On the other hand, the following aren’t instances of it:

```typescript
// ! COUNTER-EXAMPLES !
< Boo := number > 
// Structure doesn't match!
    
< Sth := < A := 1 > > 
// Expected Sth to be data type, but it was a meta object!
    
< Sth extends string > 
// Nice try, but it's just another meta type. Meta objects always specify all members! 
    
< >
// It's empty
    
< Sth = 10 > 
// Syntax error    
    
{ Sth: string }
// This is a data type.
// ! COUNTER-EXAMPLES !
```

Every member specified by a meta type must be constrained somehow. There are several possible constraints, and these constraints can reference ambient types or give types as inline expressions. For example:

```typescript
< Sth extends { complicated: Record<string, {a: 1}> } >
```

Member constraints can also reference other variables in the meta type or themselves, which allows them to represent complex webs of interacting types. For instance:

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

The same ability also lets us specify circular constraints, which are treated in the same way as circular constraints involving type parameters.

### Meta type annotations

A meta type annotation is written in the same way as a type annotation, and denotes that a member variable must be an instance of a meta type.

```typescript
meta type Example = <
    X extends unknown
>

< Sth: Example >
```

Here are some instances:

```typescript
< Sth := < X := string > >
< Sth := < X := number > >
< Sth := < X := never > >

// Once again, meta objects can also have additional structure.
< Sth := < X := string, Y := number>, Else := string >
```

Here are some non-instances:

```typescript
// ! COUNTER-EXAMPLES !

< Sth := 5 >
// Expected Sth to be a meta object, but it was a data type.
    
< Sth := < Y := string > >
// Sth doesn't have the member X.

< Sth: Example >
// This is another meta type, not a meta object.
    
// ! COUNTER-EXAMPLES !
```

### Equivalence constraint

**The equivalence constraint might end up being the most important constraint.** As we’re going to see in the usage section, it can be crucial for inference purposes.

Just as it sounds, this makes sure a type variable is *equivalent* (the precise definition will probably require the splitting of many hairs) to some construction. 

The simplest way to use this constraint looks like this:

```typescript
// This is both a meta object and a meta type
// Just like {a: "x"} is both a value and a type.
< A := string >
< A := string; B := number >
```

As you can see, the syntax is similar or identical to the syntax used for meta objects. 

That’s because it’s the meta equivalent of the relationship between the type `{a: 5}` and the value `{a: 5}`. They will always occur at different positions, so the language will never be confused about which one we mean, and there is a close correspondence between them.

That’s a silly example, though. Like other constraints, equivalence constraints can reference other type variables, and that’s the real point behind them.

```
<
	X extends unknown
	Predicate := (x: X) => boolean
>
```

Here are some instances of that meta type:

```typescript
< X := string, Predicate := (x: string) => boolean >
< X := object, Predicate := (x: object) => boolean >
< X := boolean, Predicate := (x: boolean) => boolean >
```

Here are some non-instances.

```typescript
< X := "abc", Predicate := (x: string) => boolean >
< X := "abc", Predicate := (x: string) => never >
< A := "abc", Predicate := boolean >
// Predicate is not equivalent to `(x: "abc") => boolean`
    
< Y := "abc" >
// Must have member X
    
< X := < A := 1 > >
// X must be a type    
```

## Correspondence: Generic signatures

Meta objects correspond to generic instantiations, while meta types correspond to generic signatures, albeit enriched with additional constraints.

This correspondence works similarly to a typed rest parameter in a function, except that it uses key-value semantics and not sequential semantics. 

**A generic type can define a rest type parameter with a meta type annotation describing its structure.**

```typescript
meta type Foo = <
    Foo extends unknown
	ArrayFoo := Foo[]
>
declare function something<...F: Foo>(value: F.Foo, arr: F.ArrayFoo): void;
```

As part of this integration, the `:=` constraint would be added to type parameters as well, if only for completeness’s sake.

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
meta object One = <
    A := number
>
meta object Two = <
    B := string
>
meta object Three = <
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

