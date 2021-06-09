# 可调试


<a id="c-debug"></a>
## 所有公有的类型都应该实现 `Debug` 

> All public types implement `Debug` (C-DEBUG)

这条原则几乎没有例外。


<a id="c-debug-nonempty"></a>
## `Debug` 呈现的内容永远不为空 

> `Debug` representation is never empty (C-DEBUG-NONEMPTY)

即使是概念上为空的值，其 `Debug` 呈现的内容也永远不应该是空着的。

```rust
let empty_str = "";
assert_eq!(format!("{:?}", empty_str), "\"\"");

let empty_vec = Vec::<bool>::new();
assert_eq!(format!("{:?}", empty_vec), "[]");
```
