## 索引类型

```typescript
// 这样 key 只能设置成 string | number 两种类型
interface Map<T> {
    [key: string]: T;
} 

// 当需要明确 key 的值范围需要使用下面两种方式之一
interface Book {
  [key in ("author" | "tags" | "desc")] : string
}

type Book = Record<"author" | "tags" | "desc", string>

```



## 联合类型



```typescript
const Book = {}
```



## 对象字面量的惰性初始化

解决如下错误：

```typescript
let foo = {};
foo.bar = 123; // Error: Property 'bar' does not exist on type '{}'
foo.bas = 'Hello World'; // Error: Property 'bas' does not exist on type '{}'
```

解决方案如下：

[https://jkchao.github.io/typescript-book-chinese/tips/lazyObjectLiteralInitialization.html#%E6%9C%80%E5%A5%BD%E7%9A%84%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88](https://jkchao.github.io/typescript-book-chinese/tips/lazyObjectLiteralInitialization.html#最好的解决方案)

