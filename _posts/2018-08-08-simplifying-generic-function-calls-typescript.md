---
layout: post
title:  "Simplifying Generic Function calls in Typescript"
categories: article
tags: typescript
date: 2018-08-08
---

Leveraging generics in typescript can lead to more verbose code, this can make your codebase
more difficult to read and maintain. A quick optimization you can use to reduce the
amount of syntactic noise is called _type argument inference_.

We'll use the identity function to demonstrate this feature.
In case you're wondering, an identity function is a function that returns a given argument.

Let's use this simple definition.

```typescript
const identity = <T>(arg: T): T => arg
```

Here is how we would normally call this function.

```typescript
identity<string>("hello world.")
// another example
identity<number>(123)
```

Using the _type argument inference_ feature we can simplify this code by having
typescript infer the generic based off the argument being passed in. So now both examples
become:

```typescript
identity("hello world.")
// and
identity(123)
```

Now we don't have to specify the type and get to avoid typing and reading the arrow symbols.

## Closing thoughts

This is a little optimization that improve the code clarity. While it's not worth
refactoring your whole codebase in one fell swoop, this is the perfect thing to
automate with a linter or your code editor.
