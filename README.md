# JavaScript Shorthand for accessing private fields and methods

Champion: Yehuda Katz

Author of first draft: Daniel Ehrenberg

## Summary

Within a class, to read a private field, this feature lets use `#x` instead of `this.#x`. A private method can, similarly, be called as `#x()`.

## Motivation

### Ergonomics

The proposed syntax is shorter. This shortness could be an incentive for JavaScript programmers to default to private, rather than public. Defaulting to private helps to evolve programs more easily and effectively.

With the proposed syntax for private fields and methods, the new shorthand is unambiguous and doesn't introduce any ASI hazards.

### Privacy mental model

Many people have raised concerns about private fields not being available outside the class. One possible story to explain the unavailability, rather than the `#`, is that `this.` is missing. The concept is, whenever you include `this.`, you're talking about the public surface area of the spec, but if you refer to just `#x`, it's the internal interface.

This story does not explain `other.#x` syntax (since we have class private, rather than instance private). The idea there would be, such syntax would rarely be used, and be considered a weird "metaprogramming" feature, rather than part of the normal case.

## Semantics

`#x` is exactly equivalent in semantics to `this.#x`.

That includes:
- In a method call `#x()`, `this` is the receiver passed in (not `undefined`).
- If a function literal exists inside a method, or if `Function.prototype.call` is used on a method, then whatever receiver these functions have at the time is used to look up the private field or method.
- Static fields and methods also have `this` as the receiver; for example, this means the shorthand is not so useful in referring to them in instance methods.

## Risks

A few concerns have been raised about the shorthand; these will be addressed in a future revision of this explainer to move the proposal forward.

### False lexical scoping mental model

In looking at the shorthand, some programmers have said, "oh, this is great, I understand this, it's lexically scoped shorthand!" However, that's not the semantics: the semantics are that `#x` is `this.#x`. There are a number of scenarios where the hazard could lead to the wrong intuition:
- When passing a method as a callback, where the method refers to private fields, one could expect the private field references to capture the object, but they don't. The risk to misunderstanding is possibly greatest with private methods.
- Similarly, when passing an anonymous old-style function as an argument.

Errors would be reported as a ReferenceError at runtime; JS engines could probably tell you, "undefined is not an instance of MyClass". Programmers are used to such problems, but the additional factor is that they may be confused about the lack of a textual `this`, which people have learned to look for as a hazard in callbacks.

### Method of referring to private names

None of the current proposals give a literal syntax for referring to a Private Name itself. The names are only visible through decorators. However, if such a syntax were created in the future, `#x` would be the most obvious syntax for it. If we take that syntactic space for the shorthand, it will be unavailable for referring to the private name.

### Receiver for shorthand on static private names

There are a couple possible intuitions for `#x`. One could be, "desugar into `this.#x`". Another could be, "get `#x` from the clearly appropriate receiver". For static fields and methods, the appropriate receiver would be the constructor. This is the case even if the reference happens from an instance method, or from a static method on a superclass called with a subclass as the receiver--anything but the constructor will lead to a TypeError.

The currently proposed semantics are always `this.#x`. More thought will have to go into whether the constructor is the more appropriate receiver for static private.
