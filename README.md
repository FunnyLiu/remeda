
# 源码分析

## 文件结构

``` bash
/Users/liufang/openSource/FunnyLiu/remeda
├── LICENSE
├── README.md
├── src
|  ├── _counter.ts
|  ├── _reduceLazy.ts
|  ├── _toLazyIndexed.ts
|  ├── _toSingle.ts
|  ├── _types.ts
|  ├── addProp.ts
|  ├── allPass.ts
|  ├── anyPass.ts
|  ├── chunk.ts
|  ├── clamp.ts
|  ├── clone.ts
|  ├── compact.ts
|  ├── concat.ts
|  ├── createPipe.ts
|  ├── difference.ts
|  ├── drop.ts
|  ├── dropLast.ts
|  ├── equals.ts
|  ├── filter.ts
|  ├── find.ts
|  ├── findIndex.ts
|  ├── first.ts
|  ├── flatMap.ts
|  ├── flatten.ts
|  ├── flattenDeep.ts
|  ├── forEach.ts
|  ├── forEachObj.ts
|  ├── groupBy.ts
|  ├── identity.ts
|  ├── index.ts
|  ├── indexBy.ts
|  ├── intersection.ts
|  ├── last.ts
|  ├── map.ts
|  ├── mapKeys.ts
|  ├── merge.ts - 简单的merge两个对象
|  ├── mergeAll.ts - 将多个对象值进行merge
|  ├── noop.ts - 一个只返回undefined的函数
|  ├── objOf.ts
|  ├── omit.ts - 过滤出对象中除了指定的其余项
|  ├── once.ts - 使指定函数只能执行一次，通过闭包标识变量解决
|  ├── pathOr.ts
|  ├── pick.ts - 从对象中pick某些属性
|  ├── pipe.ts
|  ├── prop.ts
|  ├── purry.ts - 一个用来给其他函数包裹的底层函数，如果参数不一致就返回返回对象而不是函数执行结果
|  ├── randomString.ts - 生成指定长度随机字符串
|  ├── range.ts
|  ├── reduce.ts
|  ├── reject.ts
|  ├── set.ts
|  ├── sort.ts
|  ├── sortBy.ts
|  ├── splitAt.ts
|  ├── splitWhen.ts
|  ├── take.ts
|  ├── takeWhile.ts
|  ├── times.ts
|  ├── toPairs.ts
|  ├── type.ts
|  └── uniq.ts
├── tsconfig.json
└── yarn.lock

```

## 外部模块依赖

请在： http://npm.broofa.com?q=remeda 查看

## 内部模块依赖

![img](./inner.svg)
  

## 逐个文件分析

### purry.ts

一个用来给其他函数包裹的底层函数，封装了对lazy参数的处理，如果参数不一致就返回返回对象而不是函数执行结果

### pick.ts

从对象找那个pick某些属性


Remeda
=============

The first "data-first" and "data-last" utility library designed especially for TypeScript.

[![Build Status](https://travis-ci.org/remeda/remeda.svg?branch=master)](https://travis-ci.org/remeda/remeda)
[![npm module](https://badge.fury.io/js/remeda.svg)](https://www.npmjs.org/package/remeda)
[![dependencies](https://david-dm.org/remeda/remeda.svg)](https://david-dm.org/remeda/remeda)

Installation
----------
```bash
npm i remeda
yarn add remeda
```
Then in .js or .ts

```js
import * as R from 'remeda'; // tree-shaking supported!
```


Why Remeda?
----------
There are no good utility libraries that work well with TypeScript. When working with Lodash or Ramda you must sometimes annotate types manually.  
Remeda is written and tested in TypeScript and that means there won't be any problems with custom typings.



What's "data-first" and "data-last"?
----------
Functional programming is nice, and it makes the code more readable. However there are situations where you don't need "pipes", and you want to call just a single function.  

```js
// Remeda
R.pick(obj, ['firstName', 'lastName']);

// Ramda
R.pick(['firstName', 'lastName'], obj);

// Lodash
_.pick(obj, ['firstName', 'lastName']);
```

In the above example, "data-first" approach is more natural and more programmer friendly because when you type the second argument, you get the auto-complete from IDE. It's not possible to get the auto-complete in Ramda because the data argument is not provided.

"data-last" approach is helpful when writing data transformations aka pipes.

```js
const users = [
  {name: 'john', age: 20, gender: 'm'},
  {name: 'marry', age: 22, gender: 'f'},
  {name: 'samara', age: 24, gender: 'f'},
  {name: 'paula', age: 24, gender: 'f'},
  {name: 'bill', age: 33, gender: 'm'},
]

// Remeda
R.pipe(
  users,
  R.filter(x => x.gender === 'f'),
  R.groupBy(x => x.age),
);

// Ramda
R.pipe(
  R.filter(x => x.gender === 'f'),
  R.groupBy(x => x.age),
)(users) // broken typings in TS :(

// Lodash
_(users)
  .filter(x => x.gender === 'f')
  .groupBy(x => x.age)
  .value()

// Lodash-fp
_.flow(
  _.filter(x => x.gender === 'f'),
  _.groupBy(x => x.age),
)(users)// broken typings in TS :(
```

Mixing paradigms can be cumbersome in Lodash because it requires importing two different methods.  
Remeda implements all methods in two versions, and the correct overload is picked based on the number of provided arguments.  
The "data-last" version must always have one argument less than the "data-first" version.

```js
// Remeda
R.pick(obj, ['firstName', 'lastName']); // data-first
R.pipe(obj, R.pick(['firstName', 'lastName'])); // data-last

R.pick(['firstName', 'lastName'], obj); // error, this won't work!
R.pick(['firstName', 'lastName'])(obj); // this will work but the types cannot be inferred

```


Lazy evaluation
----------
Many functions support lazy evaluation when using `pipe` or `createPipe`. These functions have a `pipeable` tag in the documentation.  
Lazy evaluation is not supported in Ramda and only partially supported in lodash.

```js
// Get first 3 unique values
const arr = [1, 2, 2, 3, 3, 4, 5, 6];

const result = R.pipe(
  arr,                    // only four iterations instead of eight (array.length)
  R.map(x => {
    console.log('iterate', x);
    return x;
  }),
  R.uniq(),
  R.take(3)
); // => [1, 2, 3]

/**
 * Console output:
 * iterate 1
 * iterate 2
 * iterate 2
 * iterate 3
 * /

```


Indexed version
----------
Iterable functions have an extra property `indexed` which is the same function with iterator `(element, index, array)`.

```js
const arr = [10, 12, 13, 3];

// filter even values
R.filter(arr, x => x % 2 === 0); // => [10, 12]

// filter even indexes
R.filter.indexed(arr, (x, i) => i % 2 === 0); // => [10, 13]
```

For Lodash and Ramda users
----------
Please check function mapping in [mapping.md](./mapping.md).


Remeda Design Goals
----------
1. The usage must be programmer friendly, and that's more important than following XYZ paradigm strictly.
2. Manual annotation should never be required, and proper typings should infer everything. The only exception is the first function in `createPipe`.
3. E6 polyfill is required. Core methods are reused, and data structure (like Map/Set) are not re-implemented.
4. The implementation of each function should be as minimal as possible. Tree-shaking is supported by default. (Do you know that `lodash.keyBy` has 14KB after minification?)
5. All functions are immutable, and there are no side-effects.
6. Fixed number of arguments.

MIT
