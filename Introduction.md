# Meta Types - A Proposal

**Meta Types are the natural result of applying TypeScript’s structural type system to itself**. It allows us to encapsulate complex generic signatures, construct powerful abstractions, and fulfill multiple seemingly disparate feature requests with a single type system extension.

Meta Types has two essential components:

* The **type object**, a type-level structure that embeds types like normal objects embed values.
* The **meta type**, which is the type of a type object. 

This setup is similar to type constructors (Higher-Kinded Types, HKTs, etc), but where type constructors use the language of functions to talk about types, here we’re using the language of objects to do so. 

> I discovered Meta Types while trying to implement classic HKTs in TypeScript, but I soon realized they are much more broadly applicable.

Some related features include:

* Rust’s [associated types](https://doc.rust-lang.org/rust-by-example/generics/assoc_items/types.html)
* Haskell’s type classes and HKTs.

> The meta type system has a natural extension to type constructors (Higher-Kinded Types, HKTs), which we achieve by allowing type objects to contain generic types. 
>
> This ability isn’t necessary for the feature to be useful, though, so we don’t have to support it right away. If you’re curious, take a look at my [not-quite proposal](https://github.com/microsoft/TypeScript/issues/55280), which focuses on HKTs. Some of the terminology has changed, but the core concept remains the same.

## Use cases

Meta Types is a major extension to the type system, and one that allows us to express concepts that were impossible to express before. Through this ability we can address a lot of different feature requests, as well as some core issues.

***This is an incomplete list.*** found many of these by just going through feature requests and seeing where Meta Types would be applicable. I’m sure there are even more issues that I missed.

* **Named type parameters** – Directly.
  * https://github.com/microsoft/TypeScript/issues/54254
* **Partial generic parameterization and inference** – In principle, this is implemented as type object inference. The degree to which it will happen in practice depends on implementation challenges.
  * https://github.com/microsoft/TypeScript/issues/10571
  * https://github.com/microsoft/TypeScript/issues/26242
* **HKTs** – via said extension
  * https://github.com/microsoft/TypeScript/issues/1213
* **Type display for generic types** – Meta Types allow you to use more declared elements in a generic signature, which improves type display.
  * https://github.com/microsoft/TypeScript/issues/14662
* **Simplifies generic signatures** – Meta Types encapsulate complex type parameter constraints into their own thing, which simplifies the signatures of generic types.
  * https://github.com/microsoft/TypeScript/issues/42388 – Accomplishes goal of reorganizing type constraints, as well as a variation on “scoped aliases”.
* **Self-referential types and constraints** – Type members of meta types get a self-reference token. With the HKT extension, said token is generic.
  * https://github.com/microsoft/TypeScript/issues/38038 
  * https://github.com/microsoft/TypeScript/issues/6223 – You get a generic self-reference token that you can instantiate with different types.
* **Associated types** – Given a type `A` find one or more associated types `A₁, ⋯, Aₙ`, without any kind of map that describes the association. Achieved by meta types that have `=` constraints. They let you work with entire ecosystems of related types.
  * https://github.com/microsoft/TypeScript/issues/9889
  * https://github.com/microsoft/TypeScript/issues/1758
* **Concrete types** – Can be achieved if you can express a type that proves the given relation. Might be inconvinient of type object inference doesn’t work.
  * https://github.com/microsoft/TypeScript/issues/28430
* https://github.com/microsoft/TypeScript/issues/30994 – Namespaces make more sense now!

## Design benefits

Meta Types introduce a whole new level to the type system. This may sound like overcomplicating things, but actually it has lots of benefits from a design point of view.

It’s a way of developing TypeScript’s type system horizontally, without affecting the assignability relation too much. We’re defining machinery that only affects the type world through type parameters, rather than introducing totally new types that can must be useable in all contexts.

As a consequence, it becomes natural to gradually develop the interaction of Meta Types with specific TypeScript features, instead of having to define all interactions when we introduce the system – as we would if we were introducing a regular type. 

As a specific example, we’re just not going to describe what happens if you stick meta types or type objects in conditional types – it would simply be an error. This is legitimate because meta types are special and different so you can’t just stick them wherever you want. Every such interaction could be phrased as a new feature, rather than something users just expect to work.

Meta Types can fully integrate namespaces into the type system . We’ll see how that works later.