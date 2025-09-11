---
title: Modules
layout: docs
permalink: /docs/handbook/2/modules.html
oneline: "How JavaScript handles communicating across file boundaries."
---

JavaScript는 코드를 모듈화하는 다양한 방법들의 오랜 역사를 가지고 있습니다.
2012년에 등장한 TypeScript는 이런 다양한 방법에 대한 지원을 지원해왔지만, 시간이 지남에따라 커뮤니티와 JavaScript 표준은 ES Modules(또는 ES6 모듈)이라고 불리는 형식으로 정착하게 되었습니다. 이는 `import`/`export` 구문으로 알려져 있습니다.

ES Modules는 2015년에 JavaScript 사양에 추가되었고, 2020년쯤에는 대부분의 웹 브라우저와 JavaScript 런타임에서 폭넓게 지원되기 시작했습니다.

이 핸드북에서는 집중적으로 ES Modules와 그 이전에 널리 쓰이던 CommonJS `module.exports = ` 구문을 함께 다룹니다. 다른 모듈 패턴에 대한 내용은 참고 섹션의 [Modules](/docs/handbook/modules.html)에서 확인할 수 있습니다.


## JavaScript 모듈의 정의 방법

TypeScript에서는 ECMAScript 2015와 마찬가지로 파일의 최상위<sup>top-level</sup>에 `import` 또는 `export`가 있으면 그 파일을 모듈<sup>Modules</sup>로 간주합니다.

반대로 파일 최상위에 `import` 또는 `export` 선언이 없으면 스크립트<sup>script</sup>로 취급되며, 이때 해당 내용은 전역 스코프<sup>global scope</sup>에서 참조할 수 있습니다 (모듈에서도 참조할 수 있습니다).

모듈은 전역 스코프가 아닌 자체 스코프<sup>scope</sup> 내에서 실행됩니다.
따라서 모듈 내 변수, 함수, 클래스 등이 `export` 구문으로 명시적으로 내보내지 않으면 모듈 외부에서는 참조할 수 없습니다.
반대로 다른 모듈에서 내보낸 변수, 함수, 클래스, 인터페이스 등을 참조하려면 `import` 구문을 사용해 가져와야 합니다.

## 모듈이 아닌 파일<sup>Non-modules</sup>

시작하기 전에 TypeScript가 무엇을 모듈로 간주하는지 짚고 넘어가야 합니다. JavaScript 사양에 따르면 `import` 또는 `export` 선언 또는 최상위 await가 없으면 JavaScript 파일은 모듈이 아닌 스크립트<sup>script</sup>로 간주합니다.

스크립트 파일 내에서는 변수와 타입이 공유된 전역 스코프에 선언되며, [`outFile`](/tsconfig#outFile) 컴파일러 옵션을 사용해 여러 입력 파일을 하나의 출력 파일로 합치거나, HTML에서 여러 개의 `<script>` 태그를 사용해 이 파일들을 로드합니다 (순서를 지켜야 합니다!).

`import`나 `export`가 없는 파일을 모듈로 처리하려면 다음 줄을 추가하세요:

```ts twoslash
export {};
```

위 한 줄을 추가하면 해당 파일은 아무것도 내보내지 않는 모듈이 됩니다. 이 구문은 module 타겟 설정과 관계없이 동작합니다.

## TypeScript의 모듈

<blockquote class='bg-reading'>
   <p>추가로 읽을 거리:<br />
   <a href='https://exploringjs.com/impatient-js/ch_modules.html#overview-syntax-of-ecmascript-modules'>Impatient JS (Modules)</a><br/>
   <a href='https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Modules'>MDN: JavaScript Modules</a><br/>
   </p>
</blockquote>

TypeScript에서 모듈을 사용할 때 고려할 핵심 사항은 세 가지입니다.

- **Syntax**: What syntax do I want to use to import and export things?
- **Module Resolution**: What is the relationship between module names (or paths) and files on disk?
- **Module Output Target**: What should my emitted JavaScript module look like?

### ES Module 방식

`export default`를 통해 기본 내보내기를 할 수 있습니다.

```ts twoslash
// @filename: hello.ts
export default function helloWorld() {
  console.log("Hello, world!");
}
```

이렇게 내보낸 항목들은 다음처럼 가져옵니다:

```ts twoslash
// @filename: hello.ts
export default function helloWorld() {
  console.log("Hello, world!");
}
// @filename: index.ts
// ---cut---
import helloWorld from "./hello.js";
helloWorld();
```

기본 내보내기 외에도 `default`를 생략한 `export`를 통해 변수와 함수를 여러 개 내보내기 할 수 있습니다:

```ts twoslash
// @filename: maths.ts
export var pi = 3.14;
export let squareTwo = 1.41;
export const phi = 1.61;

export class RandomNumberGenerator {}

export function absolute(num: number) {
  if (num < 0) return num * -1;
  return num;
}
```

위 항목들을 사용할 때는 `import` 구문으로 다른 파일에서 가져와 사용할 수 있습니다:

```ts twoslash
// @filename: maths.ts
export var pi = 3.14;
export let squareTwo = 1.41;
export const phi = 1.61;
export class RandomNumberGenerator {}
export function absolute(num: number) {
  if (num < 0) return num * -1;
  return num;
}
// @filename: app.ts
// ---cut---
import { pi, phi, absolute } from "./maths.js";

console.log(pi);
const absPhi = absolute(phi);
//    ^?
```

### Additional Import Syntax

An import can be renamed using a format like `import {old as new}`:

```ts twoslash
// @filename: maths.ts
export var pi = 3.14;
// @filename: app.ts
// ---cut---
import { pi as π } from "./maths.js";

console.log(π);
//          ^?
```

You can mix and match the above syntax into a single `import`:

```ts twoslash
// @filename: maths.ts
export const pi = 3.14;
export default class RandomNumberGenerator {}

// @filename: app.ts
import RandomNumberGenerator, { pi as π } from "./maths.js";

RandomNumberGenerator;
// ^?

console.log(π);
//          ^?
```

You can take all of the exported objects and put them into a single namespace using `* as name`:

```ts twoslash
// @filename: maths.ts
export var pi = 3.14;
export let squareTwo = 1.41;
export const phi = 1.61;

export function absolute(num: number) {
  if (num < 0) return num * -1;
  return num;
}
// ---cut---
// @filename: app.ts
import * as math from "./maths.js";

console.log(math.pi);
const positivePhi = math.absolute(math.phi);
//    ^?
```

You can import a file and _not_ include any variables into your current module via `import "./file"`:

```ts twoslash
// @filename: maths.ts
export var pi = 3.14;
// ---cut---
// @filename: app.ts
import "./maths.js";

console.log("3.14");
```

In this case, the `import` does nothing. However, all of the code in `maths.ts` was evaluated, which could trigger side-effects which affect other objects.

#### TypeScript Specific ES Module Syntax

Types can be exported and imported using the same syntax as JavaScript values:

```ts twoslash
// @filename: animal.ts
export type Cat = { breed: string; yearOfBirth: number };

export interface Dog {
  breeds: string[];
  yearOfBirth: number;
}

// @filename: app.ts
import { Cat, Dog } from "./animal.js";
type Animals = Cat | Dog;
```

TypeScript has extended the `import` syntax with two concepts for declaring an import of a type:

###### `import type`

Which is an import statement which can _only_ import types:

```ts twoslash
// @filename: animal.ts
export type Cat = { breed: string; yearOfBirth: number };
export type Dog = { breeds: string[]; yearOfBirth: number };
export const createCatName = () => "fluffy";

// @filename: valid.ts
import type { Cat, Dog } from "./animal.js";
export type Animals = Cat | Dog;

// @filename: app.ts
// @errors: 1361
import type { createCatName } from "./animal.js";
const name = createCatName();
```

###### Inline `type` imports

TypeScript 4.5 also allows for individual imports to be prefixed with `type` to indicate that the imported reference is a type:

```ts twoslash
// @filename: animal.ts
export type Cat = { breed: string; yearOfBirth: number };
export type Dog = { breeds: string[]; yearOfBirth: number };
export const createCatName = () => "fluffy";
// ---cut---
// @filename: app.ts
import { createCatName, type Cat, type Dog } from "./animal.js";

export type Animals = Cat | Dog;
const name = createCatName();
```

Together these allow a non-TypeScript transpiler like Babel, swc or esbuild to know what imports can be safely removed.

#### ES Module Syntax with CommonJS Behavior

TypeScript has ES Module syntax which _directly_ correlates to a CommonJS and AMD `require`. Imports using ES Module are _for most cases_ the same as the `require` from those environments, but this syntax ensures you have a 1 to 1 match in your TypeScript file with the CommonJS output:

```ts twoslash
/// <reference types="node" />
// @module: commonjs
// ---cut---
import fs = require("fs");
const code = fs.readFileSync("hello.ts", "utf8");
```

You can learn more about this syntax in the [modules reference page](/docs/handbook/modules.html#export--and-import--require).

## CommonJS Syntax

CommonJS is the format which most modules on npm are delivered in. Even if you are writing using the ES Modules syntax above, having a brief understanding of how CommonJS syntax works will help you debug easier.

#### Exporting

Identifiers are exported via setting the `exports` property on a global called `module`.

```ts twoslash
/// <reference types="node" />
// ---cut---
function absolute(num: number) {
  if (num < 0) return num * -1;
  return num;
}

module.exports = {
  pi: 3.14,
  squareTwo: 1.41,
  phi: 1.61,
  absolute,
};
```

Then these files can be imported via a `require` statement:

```ts twoslash
// @module: commonjs
// @filename: maths.ts
/// <reference types="node" />
function absolute(num: number) {
  if (num < 0) return num * -1;
  return num;
}

module.exports = {
  pi: 3.14,
  squareTwo: 1.41,
  phi: 1.61,
  absolute,
};
// @filename: index.ts
// ---cut---
const maths = require("./maths");
maths.pi;
//    ^?
```

Or you can simplify a bit using the destructuring feature in JavaScript:

```ts twoslash
// @module: commonjs
// @filename: maths.ts
/// <reference types="node" />
function absolute(num: number) {
  if (num < 0) return num * -1;
  return num;
}

module.exports = {
  pi: 3.14,
  squareTwo: 1.41,
  phi: 1.61,
  absolute,
};
// @filename: index.ts
// ---cut---
const { squareTwo } = require("./maths");
squareTwo;
// ^?
```

### CommonJS and ES Modules interop

There is a mis-match in features between CommonJS and ES Modules regarding the distinction between a default import and a module namespace object import. TypeScript has a compiler flag to reduce the friction between the two different sets of constraints with [`esModuleInterop`](/tsconfig#esModuleInterop).

## TypeScript's Module Resolution Options

Module resolution is the process of taking a string from the `import` or `require` statement, and determining what file that string refers to.

TypeScript includes two resolution strategies: Classic and Node. Classic, the default when the compiler option [`module`](/tsconfig#module) is not `commonjs`, is included for backwards compatibility.
The Node strategy replicates how Node.js works in CommonJS mode, with additional checks for `.ts` and `.d.ts`.

There are many TSConfig flags which influence the module strategy within TypeScript: [`moduleResolution`](/tsconfig#moduleResolution), [`baseUrl`](/tsconfig#baseUrl), [`paths`](/tsconfig#paths), [`rootDirs`](/tsconfig#rootDirs).

For the full details on how these strategies work, you can consult the [Module Resolution](/docs/handbook/modules/reference.html#the-moduleresolution-compiler-option) reference page.

## TypeScript's Module Output Options

There are two options which affect the emitted JavaScript output:

- [`target`](/tsconfig#target) which determines which JS features are downleveled (converted to run in older JavaScript runtimes) and which are left intact
- [`module`](/tsconfig#module) which determines what code is used for modules to interact with each other

Which [`target`](/tsconfig#target) you use is determined by the features available in the JavaScript runtime you expect to run the TypeScript code in. That could be: the oldest web browser you support, the lowest version of Node.js you expect to run on or could come from unique constraints from your runtime - like Electron for example.

All communication between modules happens via a module loader, the compiler option [`module`](/tsconfig#module) determines which one is used.
At runtime the module loader is responsible for locating and executing all dependencies of a module before executing it.

For example, here is a TypeScript file using ES Modules syntax, showcasing a few different options for [`module`](/tsconfig#module):

```ts twoslash
// @filename: constants.ts
export const valueOfPi = 3.142;
// @filename: index.ts
// ---cut---
import { valueOfPi } from "./constants.js";

export const twoPi = valueOfPi * 2;
```

#### `ES2020`

```ts twoslash
// @showEmit
// @module: es2020
// @noErrors
import { valueOfPi } from "./constants.js";

export const twoPi = valueOfPi * 2;
```

#### `CommonJS`

```ts twoslash
// @showEmit
// @module: commonjs
// @noErrors
import { valueOfPi } from "./constants.js";

export const twoPi = valueOfPi * 2;
```

#### `UMD`

```ts twoslash
// @showEmit
// @module: umd
// @noErrors
import { valueOfPi } from "./constants.js";

export const twoPi = valueOfPi * 2;
```

> Note that ES2020 is effectively the same as the original `index.ts`.

You can see all of the available options and what their emitted JavaScript code looks like in the [TSConfig Reference for `module`](/tsconfig#module).

## TypeScript namespaces

TypeScript has its own module format called `namespaces` which pre-dates the ES Modules standard. This syntax has a lot of useful features for creating complex definition files, and still sees active use [in DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped). While not deprecated, the majority of the features in namespaces exist in ES Modules and we recommend you use that to align with JavaScript's direction. You can learn more about namespaces in [the namespaces reference page](/docs/handbook/namespaces.html).
