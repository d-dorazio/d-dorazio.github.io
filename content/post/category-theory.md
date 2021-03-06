---
title: "Notes on Category Theory"
date: 2019-01-10T13:39:08+01:00
---

During the holiday season I finally decided to learn something about [Category
theory][category-theory] because it seems to be *the* hot topic in the Haskell
community. The book I used to guide me in this journey is [Category theory for
programmers][cats-for-programmers] by Bartosz Milewski.

I cannot say I fully understood it, I'm far away from doing so, but I want to
write down what I think I learned nonetheless. I hope this will clarify my
thoughts and maybe help someone else.

## What is category theory

Bartosz likes to say that Category theory is the science of structure and
composition. Since functional programmers believe, rightly imho, that programs
are nothing but a huge composition of functions it might be worthwhile to take a
look at it.

Generally speaking though, category theory is a branch of maths that tries to
define the structure of "things" by only studying the "relationships" between
them. It's closely related to [Set theory][set-theory] but the core difference
is that in category theory it's not allowed to define anything relying on
properties on the elements themselves, but only in terms of their connectivity.

## Category

A category is a bunch of objects connected by arrows. Every category has two
properties:

- every object always has an identity arrow that points to itself;
- the arrows, also called morphisms, must be composable that is if there's a
  function from object **A** to **B** and from **B** to **C** then there must be
  also an arrow from **A** to **C**. Moreover, the composition must be
  associative that is `(f . g) . h = f . (g . h)`.

If the second property sounds like the definition of function composition then
you're absolutely right! In fact, it turns out there is a category where the
objects are types and the arrows are the functions between them. In Haskell this
category is called *Hask*, but this concepts can be translated to other type
systems with almost no changes.

Here's an example of a simple category

<img src="/post/category-theory/simple-category.svg" alt="simple category" class="image-centered">

## Isomorphism

Another core concept of category theory is the concept of *isomorphism*. This
buzzword means that there's an invertible arrow, a morphism, between two objects
in a category. In other words, the two objects are isomorphic.

Simply put, this means that the composition between the two arrows is the
identity arrow. In pseudo Haskell notation this could be expressed as `f . g =
id`.

As a trivial example, let's show how `Maybe ()` is isomorphic to `Bool`.

```haskell
isJust :: Maybe () -> Bool
isJust (Just _) = True
isJust Nothing  = False

justIf :: Bool -> Maybe ()
justIf True  = Just ()
justIf False = Nothing

-- function comparison is not implemented in Haskell, but the following
-- statements shouldn't be too hard to verify
-- isJust . justIf = id
-- justIf . isJust = id
```

<img src="/post/category-theory/isomorphism-example.svg" alt="isomorphism example" class="image-centered">

Clearly, `Maybe ()` is not semantically the same as `Bool` (i.e. they're not the
same type), but they actually carry the same amount of information albeit in a
different shape.

A use case I find quite interesting for isomorphisms is in compiler
optimizations. Theoretically a compiler could substitute a data type with
another data type that is isomorphic to the original one but that's more machine
efficient.

## Duality

Since a category is just a collection of objects and arrows it's always possible
to create the dual of a category by inverting the direction of the arrows. The
dual category is also called the *opposite* category or **C**<sup>_op_</sup>.
This duality is interesting because it also means that structures built on a
category **C** also have a dual shape in **C**<sup>_op_</sup>. Usually the dual
shapes are called with the same name with a *co* prefix.

For example, here's the opposite category of first example category

<img src="/post/category-theory/simple-category-op.svg" alt="simple category" class="image-centered">

Note how the identity arrows are preserved since their source and target are the
same object.

Also note that the dual of the dual of a category is the category itself that is
(**C**<sup>_op_</sup>) <sup>_op_</sup> = **C**. This is why you can give a nut
to a category theorist even though he asked you a *coco*-nut.

## Product

At this point we have the tools to understand one of the core shapes in category
theory: the *Product*. A *product* between two objects is an object that has two
morphisms to the original objects. Intuitively, a *Product* between two objects
is another object from which it's possible to retrieve both the original
objects. In Haskell pseudo-notation these two morphisms, also called
projections, can be written as

```haskell
f :: p -> a
g :: p -> b
```

Using this definition there could be multiple valid product objects for any two
objects. To solve this ambiguity there's an additional constraint the product
must satisfy: it must have a unique morphism to any other candidate product
object. I know it sounds scary, but all it means is that the product object
cannot be built from any other candidate product object. In other words, the
product object has less structure than the other candidate products.

Here's the graph for the product **P**. Note that I've omitted the identity
morphisms for simplicity only.

<img src="/post/category-theory/product.svg" alt="product" class="image-centered">

In this diagram, there are two candidate product objects: **P** and **P'**.
However, only **P** is the product between **A** and **B** because there's a
morphism _h_ that can be used to go from **P'** to **P**.

In Haskell, the product of two types is the two element tuple `(a, b)` which can
also be expressed as `data Product a b = Product a b`. The projections are
simply the `fst` and `snd` functions.

As an example of a Haskell type that is a candidate for being the product type
let's consider `(a, b, Int)`. It's easy to show that the morphism that builds
`(a, b)` from it is

```haskell
h :: (a, b, Int) -> (a, b)
h (a, b, _) = (a, b)
```

Therefore we can say that `(a, b, Int)` is not the product of `a` and `b`.

## Coproduct

In category theory, as soon as a pattern is found, the next thing to do is to
apply the duality concept to find its dual pattern. In our case let's build the
dual of the product: the *coproduct*.

This is the product diagram with the arrows inverted.

<img src="/post/category-theory/sum.svg" alt="sum" class="image-centered">

The intuition behind this is that a *coproduct* is an object that can be built
either by **A** or by **B**. In fact, there are two morphisms, also known as
injections, that create a coproduct. Sure enough, there could be multiple
coproducts for the same pair of objects, but just like a product, the coproduct
cannot be built from any other coproduct candidate.

This might be a bit harder to recognize, but if we try hard enough we can see
the pattern of algebraic data types! In fact, in Haskell we can define the
*coproduct* in the following way

```haskell
data Coproduct a b = CoproA a | CoproB b
```

The two injections are the value constructors.

You might recognize this declaration as the declaration of the `Either` data
type and you'd be right because they're isomorphic to each other!

## Algebra

Both product and coproduct, also known as *sum*, have names of arithmetic
operations but they don't seem to relate to the concept of addition and
multiplication. Accident? As Master Oogway says, there is no accident.

As a refresher, here are the definitions of `Product` and `Sum`

```haskell
data Product a b = Product a b
data Sum a b = SumA a | SumB b
```

One of the questions we can ask is how many ways are there to create any of
these patterns.

Let's start with the product. If it had a single type parameter then the number
of values would simply be the number of distinct values in the wrapped type.
Since in the real product there are two type parameters, the total number of
ways to build a product is the product of the number of distinct values in each
type parameter!

We can apply the same reasoning to sum. To build a sum we either need an _a_ or
a _b_ but not both. Therefore the number of ways of building a sum is just the
sum of the values in the type parameters.

Now that we have operations to combine (positive) numbers it might be worthwhile
to find an encoding for the numbers themselves.

Let's start with the number one. In this line of reasoning, one means that there
should be only a single value for that type. It's not too hard to come up with a
valid definition. In Haskell something like the following will work

```haskell
data One = One
```

There's just one way of instantiating the type One that is by using the _value_
constructor One. Note how One is isomorphic to `()`.

Now that we have One we can have two, three, etc.. simply by using sums and
products! Take a look at the following type definitions

```haskell
type Two = Sum One One
type Four = Product Two Two
```

Let's check if this actually works by enumerating all the ways we can build
those types:

- we can build a Two using either `SumA One` or `SumB One`;
- a Four can be built with `Product (SumA One) (SumA One)`, `Product (SumA One)
  (SumB One)`, `Product (SumB One) (SumA One)` and `Product (SumB One) (SumB
  One)`.

It seems to work, neat!

However we haven't yet found an encoding for zero. Let's fix that now, here's
the definition

```haskell
data Zero
```

In Haskell it's possible to define a type without any value constructors. This
means that we can never create any value of this type. This is also known as the
`Void` type (which is not the same as `void` in c/c++). In C++ a type isomorphic
to zero can be defined in terms of a class with a private constructor.

Note how Zero is the identity for Sum as well as One is for Product. Also note
how any Product with Zero ends up in no ways of building that type because at
some point you will have to instantiate a Zero which is impossible.

If we substitute numbers with types it becomes evident that creating new types
is just a composition over Sums and Products (if we can create any number we can
create any type). This allows the creation of a generic framework to manipulate
data types which is how the Haskell compiler is able to implement a lot of
typeclasses automatically.

## Curry-Howard isomorphism

It's possible to show that products and sums can be transformed into constructs
in logic. In particular a product would be *logical and* or `&&` and a sum would
be a *logical or* or `||`. Furthermore, Zero would correspond to _false_ and One
to _true_.

It's quite easy to show that the familiar properties about such constructs hold.
For example `a && false = false` holds because if we translate the proposition
to sums and products it eventually evaluates to `Product a Zero` which we know
is always `Zero`. This same approach can be used to show that `a || true =
true`.

We can also show how functions can be mapped to *logical implication* or `a ->
b` meaning that if _a_ is _true_ then we can say _b_. The obvious question here
is what happens when _a_ is _false_? Well, the function would be of type `Zero
-> b` but if we look closely we might realize that we can never call that
function because Zero doesn't have any values! Therefore we cannot conclude _b_.

An interesting interpretation of this is that, since all the possible types are
a combination of products and sums and that we can map sums and products to
logical constructs then we can say that there's an isomorphism between types and
logic! This is known as the [Curry-Howard
isomorphism][curry-howard-isomorphism]. Under this point a view, checking if a
program type-checks is no different than checking whether a logical proposition
holds or not.

## Functor

A *functor* is a mapping between categories which maps both objects and
morphisms. When the target category of the functor is the same as the source
category then the functor is called *endofunctor*. In programming we don't
usually have multiple categories because we live only in the category of types
therefore in this context a functor actually is an endofunctor.

In Haskell a (endo)functor can be defined in the following way:

```haskell
class Functor f where
  fmap :: (a -> b) -> (f a -> f b)
```

The mapping between objects is in the fact that `f` must be a type constructor
that is a type that needs a single type parameter in order to be instantiated.
For example, `Maybe` and `Either String` are both type constructors because they
require only another type parameter but `Either` is not because it requires 2
type parameters. Note that also `Either String Int` or `Int` are not type
constructors because they're types and as such they require no type parameters.
We can see a type constructor **T** as taking the type parameter and mapping it
into category **T**.

If we try hard enough we can also see how `fmap` is doing the mapping between
morphisms, that is functions, from one category to another one. In fact it takes
the morphism `a -> b` which is mapped to the morphism `f a -> f b` in category
**F**.

Here's an example implementation of Functor for Maybe

```haskell
instance Functor Maybe where
  fmap f (Just x) = Just (f x)
  fmap _ Nothing = Nothing
```

Functors are really common in programming. My intuition is that they represent
some sort of containers. For example, the Maybe type can be seen as a container
of an optional value and a list can be seen as a plain container of values.

## Monoid

Another interesting pattern is the *Monoid*. A *Monoid* is an object equipped
with a unit arrow and an associative multiplication arrow.

In Haskell this could be expressed as

```haskell
class Monoid m where
  mempty :: m
  mappend :: m -> m -> m
```

Note that in Haskell mempty is not actually an arrow but a value of type `m`
because it can be shown that `m` is isomorphic to `() -> m` (there's only a
single way of calling `() -> m`).

These morphisms must satisfy the following laws

```haskell
mappend mempty m = m
mappend m mempty = m

(mappend x . mappend y) . mappend z = mappend x . (mappend y . mappend z)
```

To put it simply, mappend is an associative "multiplication-like" morphism and
mempty is the identity value under mappend.

I know this definition isn't very clear, but hopefully an example helps.
Probably the simplest example of a Monoid is the product of numbers. In fact
it's easy to show that the mempty value is 1 and that mappend is simply the `*`
operator.

Here's a possible implementation in Haskell

```haskell
data ProductM = ProductM Int

instance Monoid ProductM where
  mempty = ProductM 1
  mappend (ProductM a) (ProductM b) = ProductM (a * b)
```

The real definition is a bit different because ProductM must also be a
*Semigroup* in order to be a Monoid. Let's ignore this detail by just saying
that a Semigroup is a Monoid without the identity value; it's just the
associative morphism.

Besides the usual unwrapping and rewrapping the implementation should be quite
straightforward to follow. I won't bother demonstrating that `ProductM` indeed
satisfies the monoid requirements because I think we can all agree that it's
just multiplication.

Note how numbers form a monoid over addition too. In this case the mempty value
is 0 and mapped is simply the `+` operator.

Monoids are extremely useful in practice! Basically every container I can think
of is a Monoid under the concatenation morphism. Here's an hopefully easy to
understand implementation of Monoid for lists in Haskell

```haskell
instance Monoid [a] where
  mempty = []
  mappend = concat
```

It should be quite evident how this definition can be mapped to nearly every
container.

Also note how the implementation of Monoid for lists is totally generic over the
contained value. That's why lists are also known as *Free monoids*; they give
you a Monoid _for free_.

To show you how much monoids are useful let's solve a simple problem: find the
sum and product of a sequence of numbers in a single pass. Here's how it could
be done in Haskell

```haskell
f :: [Int] -> (Product Int, Sum Int)
f = foldl mappend mempty . map (\n -> (Product n, Sum n))

-- or using mconcat which is equivalent to the above manual foldl
-- f = mconcat . map (\n -> (Product n, Sum n))
```

Pretty sick, isn't it? The key point here is that also the tuple is a Monoid as
long as its components do!

## Comonoid

If there is a Monoid there must be also a *Comonoid* in the opposite category,
right? Indeed there is. Unfortunately I don't understand it completely.

After reversing the arrows eventually we get to a definition like the following

```haskell
class Comonoid m where
  destroy :: m -> ()
  dup :: m -> (m, m)
```

where dup must be associative and must return an exact copy of the input. In
Haskell the only possible definitions of these morphisms are

```haskell
destroy :: a -> ()
destroy _ = ()

dup :: a -> (a, a)
dup a = (a, a)
```

And that's why it's hard for me to understand. With only this possible
definition I'm not able to see a general pattern. Anyway, my intuition is if a
Monoid builds a somewhat bigger object at each mappend then the Comonoid, its
dual, must gradually destroy the already built object. I don't think that's 100%
accurate though.

It seems [that a Comonoid can be used to model linear type
systems][comonoid-linear-types] upon which many new languages are based upon.
Rust and Idris are examples of languages with a linear type system.


## Monad

It's time for the big one: the *Monad*.

> A monad is _just_ a monoid in the category of endofunctors.

If the definition doesn't say anything to you don't worry, you're not alone.
I'll try my best to explain this definition as simple as I can.

First of all, the definition says we're in the category of *endofunctors* which
means that objects in this category are endofunctors. Since a monad is a monoid
by definition, there must be an arrow that somehow multiplies two endofunctors
together. In addition, there must be a unit arrow as well. These arrows are
called *natural transformation* s. I know this is still a bit nebulous, so let's
cut it short and take a look at its definition in Haskell

```haskell
class Functor m => Monad m where
  return :: a -> m a
  join :: m (m a) -> m a
```

I think we can vaguely see how *return* is similar to mempty (especially to `()
-> m`): it takes a value and simply wraps it inside the monad. Like mempty this
can be seen as the entry point of a monad.

On the other hand, the *join* function is a bit more difficult to relate to
mappend, but the intuition is that join is actually doing the product within the
monad itself. This ends up in a flattening of the structure of the Monad.

Let's see how `Maybe` implements Monad

```haskell
instance Monad Maybe where
  return = Just

  join (Just a) = a
  join Nothing = Nothing
```

However, in Haskell the monad is not defined in terms of join but instead in
terms of the *bind* operator or `>>=`. It's possible to show that bind can be
defined in terms of join. Here's how it could be done

```haskell
(>>=) :: Monad m => (a -> m b) -> m a -> m b
f (>>=) m = join . fmap f $ m
```

However, I don't think this categorical definition of a monad explains very well
what a monad is in the context of programming. My intuition for the monad is
that it's a context inside which we can encapsulate values and compose functions
without ever leaving the context. For example, in the case of Maybe the context
is uncertainty because we don't know for sure whether a value is present or not.
Every function that works on a Maybe inherits this uncertainty because it must
remain in context.

## Comonad

This is the last pattern we'll talk about: the *Comonad*, the dual of the Monad.

If we try to reverse the definition of a monad we eventually get to the
following Haskell definition of a comonad

```haskell
class Functor m => Comonad m where
  extract :: m a -> a
  duplicate :: m a -> m (m a)
```

If you look at it long enough you might see similarities to the Comonoid. In
fact

> A comonad is _just_ a comonoid in the category of endofunctors

I think I understand the comonad more than the comonoid even though the former
is defined on the latter. My intuition is that, if the monad provides a way of
encapsulating a value inside a context without providing a way of accessing it
outside the context then the comonad must be all about extracting values from a
context. The *extract* function should reflect this logic quite clearly. On the
other hand, *duplicate* is the exact opposite of join because it adds another
layer of nesting in the comonad.

## Yoneda lemma

I've talked enough about category theory under a mathematical point of view.
Maths has always been connected to philosophy from its origins and sure enough
category theory is no exception.

In particular, I'd like to mention the Yoneda lemma which (semantically) says
that

> Every object is completely identified by the morphisms towards the other
> objects.

I have not understood what it actually means in category theory, but I find it
quite fascinating when applied to the category of the real world. In this
category, where objects correspond to people and arrows correspond to the
relationships between them, the Yoneda lemma means that each one of us is
uniquely defined by our relationships. I don't think that's too far away from
the truth.

## Haskell point free notation

During my journey in category theory, I also learned why the point free notation
is so named in Haskell. If you don't know what it is it's a style of writing
functions where arguments are not explicitly mentioned. The main tool to achieve
this is by using function composition. Here's an example

```haskell
oddSquares :: [Int] -> [Int]
oddSquares = map (* 2) . filter odd
```

As you can see, this function takes a single parameter, a list of numbers, which
is not mentioned explicitly in the function definition.

It's so named because arguments can also be called points in the sense of
*calculating the function at the given points*. The fact that the function
composition operator is a point too is an unfortunate coincidence.

## Conclusions

Category theory has way more concepts than what I tried to describe here and I
didn't understand most of them. For example, I didn't get T-Algebras,
Adjunctions, Kan extensions, the Lawvere theory and many more!

My main problem with category theory is that I cannot relate to most of its
concepts. In other words, I'm not able to understand what I cannot use. I want
to improve with regard to this, but it's hard.

However, I think that even with this tiny subset of category theory the inner
structure of a lot of problems can be revealed. Hopefully it also helps building
well structured software.


[category-theory]: https://en.wikipedia.org/wiki/Category_theory
[cats-for-programmers]:
https://bartoszmilewski.com/2014/10/28/category-theory-for-programmers-the-preface/
[set-theory]: https://en.wikipedia.org/wiki/Set_theory
[curry-howard-isomorphism]: https://en.wikipedia.org/wiki/Curry%E2%80%93Howard_correspondence
[comonoid-linear-types]: https://stackoverflow.com/questions/23855070/what-does-a-nontrivial-comonoid-look-like
