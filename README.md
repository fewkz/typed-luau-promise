# Typed Roblox Promises

This is a wrapper that adds full Luau type support for
[roblox-lua-promise](https://github.com/evaera/roblox-lua-promise). It does this
using some fancy Luau type trickery. The result is a way more pleasant
experience when using Promises in luau where everything is fully typed, even
when you chain promises.

## Installation

To install, simply copy the Promise.lua script into your project and modify the
`game.ReplicatedStorage.Packages.UntypedPromise` at the bottom with the path to
roblox-lua-promise.

## Usage

You can simply use this typed wrapper the same way you'd use
`roblox-lua-promise` itself. This is a drop-in replacement that you will
immediately reap the benefits of in your existing code.

## What's so special about this?

This wrapper fully supports chaining promises, which most other wrappers are
unable to. It does this by repeating the type definition of Promise multiple
times. This is because recursive types are not supported in Luau, making this
neccessary to have a correct definition for promises.
![image](https://github.com/fewkz/typed-luau-promise/assets/83943819/8fec9389-1ca3-407b-ae0e-b2dc19278fdd)

## Why is there a generation script?

Instead of having to manually write out a repeated type definition 10 times,
there's a `generate-promise.ts` script that does this for us, making it very
easy to modify the definition without having to make the change multiple times.
To run the generation script, simply do
`deno run generate-promise.ts > Promise.lua`

## Things you may run into

### Incomplete definition

This wrapper doesn't define every single function in roblox-lua-promise because
I never got around to it. Adding new ones is relatively straight forward, and
may be added whenever I run into the need. Feel free to contribute a pull
request adding any missing definitions.

### Promise.all

The definition of `Promise.all` requires all of the promises passed in to have
the same non-variable return type. This is so that the output promise can be
typed as `Promise<{ T }>`. It will cause issues if you have code like this:

```lua
local pA = fetchA() -- returns `string`
local pB = fetchB() -- returns `number`
local pC = fetchC() -- returns `boolean`
local a, b, c = unpack(Promise.all({pA, pB, pC}):expect()) -- errors since promise types aren't uniform
```

You should ideally rewrite the code to look this instead:

```lua
local pA = fetchA()
local pB = fetchB()
local pC = fetchC()
local a = pA:expect()
local b = pB:expect()
local c = pC:expect()
```

### Generic variable type

For cases where a variable may need to store a generic promise, you can use the
`AnyPromise` type. This stores any promise, however you don't get any type info
about what the promise will return.

```lua
local p: Promise.AnyPromise

p = Promise.resolve("hello")
p = Promise.resolve(5)
p = p:andThen(function(r) -- r is typed as `any`
    return r
end)
```
