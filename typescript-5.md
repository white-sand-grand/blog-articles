---
title: TypeScript 5.0 新特性详解
description: 深入了解 TypeScript 5.0 带来的创新功能
pubDate: 2026-07-18
category: learning
tags: ['TypeScript', 'JavaScript', '类型系统']
draft: false
---

# TypeScript 5.0 新特性详解

TypeScript 5.0 带来了装饰器、const 类型参数等重要更新。

## 装饰器

```typescript
function logged(target: any, context: ClassMethodDecoratorContext) {
  console.log(`${context.name} 被调用了`)
}

class Example {
  @logged
  greet() {
    console.log('Hello!')
  }
}
```

## const 类型参数

```typescript
function pair<const T, const U>(a: T, b: U): [T, U] {
  return [a, b]
}

const result = pair([1, 2], [3, 4])
// 类型是 [1, 2], [3, 4]，不仅仅是 number[]
```

## 总结

TypeScript 持续进化，为开发者提供更好的开发体验。
