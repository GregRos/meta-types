# Other feature requests

###   TanStack

```
export meta type Functions := <
	FQueryKey extends QueryKey
	GetPreviousPage := (...) => unknown
	

// Identify common type parameters
// Identify derived types from these parameters. For example:
export meta type Result := <
	Data: type
	Error: type
	Base := {
		// Put QueryObserverBaseResult<...> here
	}
	Loading := Base & {
		// Put QueryObserverLoadingResult<...> here
	}
	// ... all the other SomethingResults.
>

// If there are other result types with more parameters
// Use conjunction
export meta type MutationObserverResult := Result & <
	// These are extra parameters:
	Variables = void
	Context: type
	
	// Here you can access both Result and this meta type
	Base := {
		// ... base result goes here
	}
>
	

// The Query types have a superset of parameters
// Including those of the Results, so we use composition.
// You can access the different results via Result.Loading etc
export meta type Query := <
	QueryFnData: type
	// Composition using the Result meta type:
	Result: Result
	QueryKey extends QueryKey
	QueryData: type = QueryFnData
	
	Options := {
		// Put QueryOptions<...> here
	}
	
	ObserverOptions := Options & {
		// Put QueryObserverOptions<...> here
	}
	
	
>

declare function useQuery<Q: UseQuery>
```



## Zod

```
export interface ZodTypeDef {
  errorMap?: ZodErrorMap;
  description?: string;
}

export meta type ZodObjectDef := <
	T extends ZodRawShape
	UnknownKeys extends ZodTypeAny = ZodTypeAny
	Catchall extends ZodTypeAny = ZodTypeAny
	Output = 

export meta type ZodType := <
	Output: type = any
	Def extends ZodTypeDef
	
	
1
export class ZodArray<T:ZodCore, Cardinality = "many"> {
	
}

export meta type ZodInOut := <
	Output = any
	Def: ZodTypeDef
	Input = Output
>

export class ZodType<Z: ZodInOut> {
	
}

export meta object ZodString := <
	

export class ZodString extends ZodType<>





export meta type Zod := <
	Output: type = any
	Def extends ZodTypeDef
	UnknownKeys extends UnknownKeysParam
	Catchall extends ZodTypeAny = ZodTypeAny
	Input: type = any
	Object := {
		// You would implement
	}
	
```







So due to the large number of

## Named/keyed type parameters

* https://github.com/microsoft/TypeScript/issues/54254
* https://github.com/microsoft/TypeScript/issues/38913
* https://github.com/microsoft/TypeScript/pull/23696

Meta types let you express this directly.

## Organizing generic signatures

* https://github.com/microsoft/TypeScript/issues/42388

Address directly.

This is one of the original motivating examples. Something like this has been attempted at various points in TypeScript’s history

In truly meta fashion, I’m going to show how other feature requests can be expressed using meta types, which automatically covers their use cases as well.





While meta types are super cool, it might be hard to see how they can be integrated into normal TypeScript code. 

Well, there’s basically three ways.

1. Namespaces
2. Modules
3. ???

Let’s take a look at them

## Namespaces

As we’ve discussed, namespaces declare meta objects (in addition to everything else). As such, they can have a meta annotation. This meta annotation acts similarly to an `implements` specifier, but it only applies to types.

A namespace with a meta annotation must export all variable members. Members with equivalence can be exported as well, but they must not declare public members that 



```
meta type Bar := <
	A extends string
	B extends number
	Complex extends {
		add(other: Complex)
		sub(other: Complex)
	}
>
namespace Foo: Bar {
	export type A = "abc"
	export type B = 4
	export class 
}
```



Meta types are very far away from runtime code. There are essentially two type systems between them and JavaScript.

Well, there’s three ways. Let’s investigate 

1. Namespaces
2. Modules
3. ???







That was a lovely story, right? But it’s probably not worth that much without some usage examples. 

In practice, 

1. The value problem.
2. The context problem.
3. 

**The meta type system is integrated into the rest of the language through meta annotations on type parameters.**

### Context problem

Meta types 

We’ve actually talked about meta annotations already – they’re used in meta types to constrain meta object members. The same system is used to integrate meta types into regular types.

I haven’t really shown a complete usage of this feature, so here it is.

```
meta type VectorSpace := <
    Scalar extends {
        add(x: Scalar): Scalar
        sub(x: Scalar): Scalar
        mult(x: Scalar): Scalar
        div(x: Scalar): Scalar
    }
    Vector extends Iterable<Scalar> & {
        get(i: number): Scalar
        add(x: Vector): Vector
        scale(x: Scalar): Vector
    }  
    Space := {
    	vector(...scalars: Scalar[]): Vector
    	readonly zero: Scalar
    	readonly unit: Scalar
    }
>

namespace Cn: VectorSpace {
	export class Scalar = class Complex {
		
	}
	export class Vector {
		
	}
	export const Space = {
		vector(...scalars: Scalar[]): Vector
		
	}
}



meta object ABC: VectorSpace = <
	Scalar := Complex

export function span<VS: VectorSpace>(basis: VS.Basis) {
	
}

class Complex implements VectorSpace["Scalar"] {
	// 
}

const vsd = new VectorSpace([
	
])


```

Meta annotations on type parameters work similarly to type annotations on value parameters and will, hopefully, reuse the same machinery. While they don’t behave differently from other type parameters in principle, in practice they probably will.

## Type checking



## Inference

In all cases of inferring constrained type parameters, the compiler needs to construct a set of types matching said constraints, as well as the types of the supplied parameters.

This is akin to solving a set of equations involving relations between types. If we are given f

### Failure to construct

**It may not be possible for inference to construct an instance of a meta type.** This means that a function such as `foo` above might not be callable in all typing environments using `foo()`, as a function with normal type parameters would be.

I don’t necessarily view this is a downside. In many cases, TypeScript’s aggressive inference of type parameters can cause unexpected issues in other parts of the code. 

Because meta types are complex, self-referential, and possess arbitrary structure as well as complicated constraints, **it may not be possible for inference to construct an instance of one.** This may be because the meta type isn’t inhabited and we missed it, but it might 

This means that 



So far we’ve discussed meta types in the abstract, as a self-contained system. But there is another aspect – the way it’s integrated into the rest of the language. This is done through **meta annotations.**

