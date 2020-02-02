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

