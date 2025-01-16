# @denis_bruns/data-reflector

> **A JSONPath-based data transformation library for clean architecture projects, allowing you to define flexible, typed “reflectors” for mapping complex input structures to new output formats.**

[![NPM Version](https://img.shields.io/npm/v/@denis_bruns/data-reflector?style=flat-square&logo=npm)](https://www.npmjs.com/package/@denis_bruns/data-reflector)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0-blue?style=flat-square&logo=typescript)](https://www.typescriptlang.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](https://opensource.org/licenses/MIT)
[![GitHub](https://img.shields.io/badge/GitHub--181717.svg?style=flat-square&logo=github)](https://github.com/h3llf1r33/data-reflector)

---

## Overview

`@denis_bruns/data-reflector` makes it easy to **transform** or **map** data from one shape to another using **JSONPath** expressions, custom functions, or nested object mappings. It integrates well with clean architecture setups by letting you define typed **`reflectors`** that clearly express how to derive output fields from an input object.

Key capabilities include:
- **Type-safe** transformations via `DataReflector` definitions.
- **JSONPath** support for querying nested or array-based structures.
- **Functional** or **object-based** extractors for advanced transformations.
- **Circular reference** detection for safer data handling.

---

## Key Features

1. **Flexible Extractors**
    - Use **JSONPath** strings (e.g., `"$['data'][*]['mapping']['test']['value']"`) to select values.
    - Or provide **functions** that accept the entire input object for custom transformations.
    - Or **nested object reflectors** if you need to recursively build complex output objects.

2. **Circular Reference Detection**
    - Ensures that if the data you’re reflecting contains circular references, an error is thrown before you accidentally blow up your application.

3. **Typed & Extensible**
    - Leverages `DataReflector<Input, Output>` to ensure each output field is typed.
    - Compatible with other parts of your domain model, letting you maintain **clean architecture** boundaries.

4. **Integration with `@astronautlabs/jsonpath`**
    - JSONPath queries can be as simple or advanced as you need (`$..`, `?()`, `[*]`, etc.).

5. **Tested & Reliable**
    - Supports edge cases like empty arrays, deeply nested objects, custom function errors, and more.

---

## Installation

Install via **npm**:

```bash
npm install @denis_bruns/data-reflector
```

Or via **yarn**:

```bash
yarn add @denis_bruns/data-reflector
```

You’ll also want to ensure `@astronautlabs/jsonpath` is installed (it is a dependency but not a peer dependency):

```bash
npm install @astronautlabs/jsonpath
```

---

## Basic Usage

Here’s a quick example that demonstrates the **reflect** function using JSONPath and inline functions:

```ts
import { reflect } from "@denis_bruns/data-reflector";
import { DataReflector } from "@denis_bruns/web-core-ts";

interface InputType {
  user: {
    name: string;
    nested: { count: number }[];
  };
  timestamp: number;
}

interface OutputType {
  username: string;
  bigCount: number;
  timeString: string;
}

const input: InputType = {
  user: {
    name: "Alice",
    nested: [{ count: 1 }, { count: 10 }, { count: 100 }],
  },
  timestamp: 1704067200,
};

// Define your DataReflector
const exampleReflector: DataReflector<InputType, OutputType> = {
  // 1) JSONPath extraction
  username: "$.user.name",

  // 2) Nested function transform
  bigCount: (inp) => {
    // Summing nested counts, then doubling
    const counts = inp.user.nested.map((item) => item.count);
    const sum = counts.reduce((acc, c) => acc + c, 0);
    return sum * 2; // e.g., 1+10+100=111, times 2 => 222
  },

  // 3) Another function
  timeString: (inp) => new Date(inp.timestamp * 1000).toUTCString(),
};

const result = reflect(exampleReflector, input);
console.log(result);
/*
{
  username: 'Alice',
  bigCount: 222,
  timeString: 'Fri, 30 Dec 2023 00:00:00 GMT' 
}
*/
```

### Handling Arrays & Complex Paths

You can also query arrays with JSONPath:

```ts
const arrayReflector: DataReflector<InputType, { allCounts: number[] }> = {
  allCounts: "$.user.nested[*].count",
};

const arrayResult = reflect(arrayReflector, input);
console.log(arrayResult.allCounts); // [1, 10, 100]
```

### Handling Circular Data

If your data has a **circular** reference, the library will throw an error:

```ts
const objA: any = { name: "A" };
const objB: any = { name: "B" };
objA.ref = objB;
objB.ref = objA;

const inputWithCycle = { user: objA, timestamp: 1704067200 };

const cycleReflector: DataReflector<any, { data: any }> = {
  data: "$.user",
};

// This will throw an error due to circular data
reflect(cycleReflector, inputWithCycle);
```

---

## Advanced Features

1. **Inline Functions**
    - Provide a function `(input) => {...}` anywhere in your reflector for advanced logic.

2. **Nested Reflectors**
    - Instead of a JSONPath or function, supply another object literal that itself uses JSONPaths or functions.

3. **Wildcard Queries**
    - Query multiple array elements with `[ * ]`, or use advanced filters like `[?(@.count > 50)]`.

4. **Error Handling**
    - If your custom function throws an error, the process stops immediately, letting you handle or log the issue.

---

## Related Packages

- **@denis_bruns/web-core-ts**  
  [![NPM](https://img.shields.io/npm/v/@denis_bruns/web-core-ts?style=flat-square&logo=npm)](https://www.npmjs.com/package/@denis_bruns/web-core-ts)  
  [![GitHub](https://img.shields.io/badge/GitHub--181717.svg?style=flat-square&logo=github)](https://github.com/h3llf1r33/web-core-ts)  
  *Contains the `DataReflector` and `DataReflectorValue` interfaces that power this library.*

---

## Contributing

Contributions, issues, and feature requests are welcome! Feel free to submit a PR or open an issue on [GitHub](https://github.com/h3llf1r33/data-reflector).

---

## License

This project is [MIT licensed](LICENSE).

---

<p align="center">
  Built with ❤️ by <a href="https://github.com/h3llf1r33">h3llf1r33</a>
</p>