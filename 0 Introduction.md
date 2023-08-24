# Introduction

It’s common for mainstream libraries to have unreadable generic signatures and instantiations. This is something we see everywhere — client or server-side, libraries for any purpose or goal. Here is an example from [zod](https://github.com/colinhacks/zod):

```typescript
export class ZodObject<
  T extends ZodRawShape,
  UnknownKeys extends UnknownKeysParam = UnknownKeysParam,
  Catchall extends ZodTypeAny = ZodTypeAny,
  Output = objectOutputType<T, Catchall, UnknownKeys>,
  Input = objectInputType<T, Catchall, UnknownKeys>
> extends ZodType<Output, ZodObjectDef<T, UnknownKeys, Catchall>, Input> {
    // ...
}
```

Many other examples are referenced in #54254, each one more complicated than the next. Library developers have come up with various ways of managing these. One common method, usually called a “generic bag,” goes something like this:

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

Which, through the problems associated with inferred types, added new complexity to something that was already bordering on the unmanageable.

But let’s say you successfully finish your construction, add the necessary unit tests, and deploy your package. The story is still far from over — because now, should users of your package step out of line, your monstrous generic creations are thrust upon them as terrifying error messages that haunt their waking dreams and nightmares alike.

## Not an easy problem to solve

This isn’t an easy problem to solve — if it was, the solution would already be here, if only based on the demand for it.

The problem, as I see it, is this:

> TypeScript’s structural nature is ill-suited to work with type parameters, which don’t actually correspond to any structure at all, but instead are treated almost like rewriting rules that exist outside of the type system.

This same issue affects most proposed features dealing with type parameters in some way. My original interaction with it was through the lens of [higher-kinded types](https://github.com/microsoft/TypeScript/issues/1213), which have been suggested since the language’s inception. I even participated in some of the discussions, [back in the day](https://github.com/microsoft/TypeScript/issues/1213#issuecomment-372112121). So I’ve actually been chewing on this for a few years now.

I opened an issue about this topic a few weeks ago. It was focused on HKTs (#55280), but I’ve since changed my thinking. What I thought was a cool way of implementing HKTs was actually something more – a way for TypeScript to reason about its generics system in general.

**In fact, this proposal doesn’t express HKTs**, and instead views them as an extension that could be developed in the future. This is mainly because I feel things are complicated enough as it is.

Anyway, that’s enough for the intro.

# The Meta Type System

**The meta type system is the result of applying TypeScript’s structural type system to itself**. It allows us to encapsulate complex generic signatures, construct powerful abstractions, fulfill multiple feature requests with a single type system extension.

The meta type system has two components:

* The **meta object**, a type-level structure that embeds types like normal objects embed values.
* The **meta type**, which is the type of a meta object. 

The meta type system is a unique extension designed specifically for TypeScript, and based on existing TypeScript concepts, rules, and conventions. While it resembles features found in other languages, like Haskell and Scala, it’s very much its own thing, and it wouldn’t make sense anywhere else.

However, that doesn’t mean we can’t make some analogies. Here is the first one:

> The meta type system is like Haskell’s higher-kinded types or type constructors. Where Haskell uses the language of functions to talk about types, here we use the language of objects to do so.

## Use cases

Meta Types is a major extension to the type system, and one that allows us to express concepts that were hard to express before. As such it addresses lots of different feature requests.

***This is an incomplete list.*** Meta types is a complex and powerful system, and the way in which it can be used isn’t obvious. So there could be tons of other use cases out there.

| Req    | Name                             | How it’s addressed                                           |
| ------ | -------------------------------- | ------------------------------------------------------------ |
| #54254 | Named type parameters            | Directly, there is also a shorthand                          |
| #26242 | Partial generic parameterization | Meta object inference                                        |
| #17588 | Associated types                 | Via implicit structure, but classes don’t support it directly |
| #14662 | Generic type display             | Generic signatures can be presented as declared entities     |
| #42388 | `where` clause                   | Solves the same issues, including “scoped aliases”           |
| #39526 | Overloading generic signatures   | Disjunction meta types                                       |
| #54157 | Same class, different generics   | Disjunction meta types + implicit structure                  |
| #1213  | HKTs                             | Via HKT extension                                            |

Practical examples can be found at the end of this issue.

