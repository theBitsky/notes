---
title: 'ts-node'
tags: ['typescript', 'node']
public: true
date: '2021-05-11'
---

# ts-node

A tool that executes a [[Node]] program that is written in [[TypeScript]]

Basically, instead of using this command every time:

```
tsc index.ts && node index.js
```

you can use this tool.

### Installing and Usage

Install it globally by [[npm]]:

```
npm install -g ts-node
```

Create a file *add.ts* with simple [[TypeScript]] code:

```
function add(a: number, b: number): number {
  return a + b;
}

console.log(add(5, 5));
```

Run **ts-node** to execute this file:

```
ts-node add.ts
```

It will print the result:

```
10
```


### Links

- [ts-node](https://github.com/TypeStrong/ts-node)