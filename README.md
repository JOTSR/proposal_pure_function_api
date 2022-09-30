# Pure Function API

## Status

**Stage:**

**Author:** [JOTSR](https://github.com/JOTSR)

**Champions:**

## Overview / Motivation

This API provides a way to check if a function is pure and thus allow us to fearless compose them or paralellize them in thread pools. This also assure a way to make future high level API that can use secure and context free function that can run beside main script.

This API is based on at least one of the following proposals :
- [Explicit Ownership Syntax](https://github.com/JOTSR/proposal_explicit_ownership_syntax)
- [Explicit Reference Syntax](https://github.com/JOTSR/proposal_explicit_reference_syntax)

## API

```ts
interface Function {
    isPure(func: Function): boolean
}
```

isPure return true if the passed function only use arguments that respect (or use 0 arguments):
- "@arg" (read only reference, [see](https://github.com/JOTSR/proposal_explicit_reference_syntax))
- "copy arg" (deep copied argument, [see](https://github.com/JOTSR/proposal_explicit_ownership_syntax))
- "move arg" (scope moved argument, [see](https://github.com/JOTSR/proposal_explicit_ownership_syntax))

And all function used in current function body return true for Function.isPure()

## Examples

## No arguments

```ts
function doNothing() {
    return
}

console.assert(Function.isPure(doNothing) === true)
```

## Read-only reference arguments
```ts
function compute(@matrixA, @matrixB) {
    const result = Matrix.add(@matrixA, @matrixB)
    console.log(result)
    return result
}

console.assert(Function.isPure(compute) === false)
```

## Explicit onwnership described arguments
```ts
async function extractUser(copy username) {
    const datas = await fetch('api/username')
    return await datas.json()?.user
}

console.assert(Function.isPure(extractUser) === false)

const datas = await fetch('api/username')
async function extractUserPure(move datas) {
    return await datas.json()?.user
}

console.assert(Function.isPure(extractUserPure) === true)
```

## Arguments mix
```ts
function foo(@arg0, copy arg1, move arg2) {
    return result
}

console.assert(Function.isPure(foo) === true)

function buzz(@arg0, copy arg1, move arg2, arg3) { //arg3 is no referenced of ownerhip described argument
    return result
}

console.assert(Function.isPure(foo) === false)
```

## FAQs
- Throw an expection can be a criteria of non-pure function in order to assure the constant execution flow.