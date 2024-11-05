# safe-navigation

## Summary

Introduce an operator to short circuit expressions where a value is nil using new syntax

## Motivation

Luau currently has no way to concisely perform long access chains/operations on results of these chains where an index may result in a nil value prior to the last index.

## Design

To solve this, I propose implementing 3 new pieces of syntax.
For safe navigation with a identifier, the `?.` operator, for value indexing the `?[]` operator, and for method access the `?:` operator.

These operators will short circuit expressions which follow it to nil if the index operation returns a nil value.

Example:

```luau
-- the local baz will be nil if foo is nil OR if baz does not exist in bar (as with normal indexing)
local baz = foo?.bar.baz

-- the local baz will be nil if foo is nil OR if bar is nil OR if bar[1] is nil (as with normal indexing)
local baz = foo?.bar?[1]

-- bar:doSomething() will not be called if foo is nil
foo?.bar:doSomething()
```

This short circuiting will be limited to its simpleexp, for example:

```luau
-- the simpleexp a?.b() is subject to the short circuiting logic but the == comparison is not, neither is the rest of the ternary-if expression
local is_zero = if a?.b() == 0 then "zero" else "not-zero"
```

In order to remedy an issue previously encountered in a similar RFC, indexing operation will invoke __index with a third boolean operand indicating
if the index is to be performed in a "safe" way.

Example:
```luau
local t = setmetatable({}, {
  __index = function(t, i, safe)
    return if safe then "safe" else "non-safe"
  end
})

assert(t.foo, "non-safe")
assert(t?.foo, "safe")
```

## Drawbacks

This feature has been rejected before due to its interaction with the Roblox DOM implementation.

Though the `__index` argument will allow integration with the Roblox DOM without adding another metamethod it requires additonal VM support to handle passing the safe index operator into the __index metamethod.

## Alternatives

### Do nothing
This is simply a way to more conscisely access operate on the results of indicies, although this implementation would enable some things which cannot be implemented in current Luau without use of functions and returns

Example:
```luau
-- this is equivalent to foo?.bar?.baz
local baz = if foo and foo.bar then foo.bar.baz else nil

-- this is NOT equivalent to foo?.bar() where foo?.bar() returns multiple values as the ternary if can only be bound to one variable
local one, two = if foo and foo.bar then foo.bar() else nil
```
