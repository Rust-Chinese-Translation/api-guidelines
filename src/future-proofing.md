# 前瞻性


<a id="c-sealed"></a>
## 封装的 traits 隔绝下游的实现 

> Sealed traits protect against downstream implementations (C-SEALED)

有些 traits 仅仅在定义它们的 crate 里使用。
这时，我们依然能通过封装 trait 的方式修改 trait 代码，
从而不破坏现有的其他代码。

```rust,ignored
/// 这个 trait 被封装起来了，当前 crate 之外无法给类型实现 `private::Sealed`
pub trait TheTrait: private::Sealed {
    // 给使用者使用的方法，数量可以是 0 或更多
    fn ...();

    // 不给使用者使用的方法，数量可以是 0 或更多
    #[doc(hidden)]
    fn ...();
}

// 给类型实现 `TheTrait`
impl TheTrait for usize {
    /* ... */
}

mod private {
    pub trait Sealed {}

    // 给相同的类型实现 `Sealed` ，别的类型不实现 `Sealed`
    impl Sealed for usize {}
}
```

`Sealed` 是没有方法的、私有的、`TheTrait` 的父 trait (supertrait) ，
下游 crates 无法与 `Sealed` 有一样的命名（因为绝对路径），从而保证了 `Sealed` 的实现
（也是 `TheTrait` 的实现） 只存在于当前 crate 。
我们能自由地给 `TheTrait` 添加方法，这种 “非破坏性” 的修改，
倘若在未封装 traits 的情况下 通常是重大的变更。
此外，我们能自由地修改不被公开说明的方法签名（这不是私有的）。

注意，在封装的 trait 里 移除公开的方法或者改变公开方法的签名
仍然是破坏性的修改。

为了避免打消使用 trait 的人积极性，封装的 trait 和当前 crate 之外不能实现的 trait 
应该用 rustdoc 加以说明。

例子：

- [`serde_json::value::Index`](https://docs.serde.rs/serde_json/value/trait.Index.html)
- [`byteorder::ByteOrder`](https://docs.rs/byteorder/1.1.0/byteorder/trait.ByteOrder.html)


<a id="c-struct-private"></a>
## 结构体具有私有字段 

> Structs have private fields (C-STRUCT-PRIVATE)

让结构体字段公开，这是一个强有力的承诺：
确定了选择什么来呈现给使用者，
并且让结构体 免于提供各种验证，也不必保持字段内容不变，
因为使用者可以任意修改结构体的公开的字段数据。

公开的字段最适合 C 语言风格的结构体：
复合、被动的数据结构（[PDS]）。
除此之外的场景，请考虑隐藏字段，然后提供 getter/setter 方法。

[PDS]:https://en.wikipedia.org/wiki/Passive_data_structure

<a id="c-newtype-hide"></a>
## newtypes 封装起实现的细节 

> Newtypes encapsulate implementation details (C-NEWTYPE-HIDE)

newtype 在隐藏实现细节的同时，还可以给使用者提供精简而确切的代码。

比如，下面的 `my_transform` 函数返回复合的迭代器类型。

```rust,ignored
use std::iter::{Enumerate, Skip};

pub fn my_transform<I: Iterator>(input: I) -> Enumerate<Skip<I>> {
    input.skip(3).enumerate()
}
```

如果想要向使用者隐藏这个类型 —— 从使用者的角度看返回的类型是粗略的
`Iterator<Item = (usize, T)>` ，那么可以使用 newtype ：

```rust,ignored
use std::iter::{Enumerate, Skip};

pub struct MyTransformResult<I>(Enumerate<Skip<I>>);

impl<I: Iterator> Iterator for MyTransformResult<I> {
    type Item = (usize, I::Item);

    fn next(&mut self) -> Option<Self::Item> {
        self.0.next()
    }
}

pub fn my_transform<I: Iterator>(input: I) -> MyTransformResult<I> {
    MyTransformResult(input.skip(3).enumerate())
}
```

除了简化函数签名， newtype 还向使用者提供更少的细节代码。
用户不知道返回结果的迭代器是如何构造和呈现的，
这意味着将来可以在不破坏用户代码的情况下，修改这处封装的部分。

Rust 1.26 引入了 [`impl Trait`] 语法，
这比 newtype 模式更简明，但也带来一些缺点，
即你会被限制在 `impl Trait` 的表达里 而不够自由。
比如在返回实现了 `Debug` 或 `Clone` 或组合其他 trait 的迭代器类型时会遇到麻烦。
总之， `impl Trait` 用在返回值类型上，在内部 APIs 里使用它或许很不错，
甚至在公开的 APIs 里使用它更合适，然而并不是所有场合都合适。
参考 *版本指南* 的 ["`impl Trait` for returning complex types with ease"][impl-trait-3]
部分 和 [`impl Trait` 发行记录][impl-trait-3] 来找到更多细节。


[`impl Trait`]: https://github.com/rust-lang/rfcs/blob/master/text/1522-conservative-impl-trait.md
[impl-trait-2]: https://doc.rust-lang.org/edition-guide/rust-2018/trait-system/impl-trait-for-returning-complex-types-with-ease.html
[impl-trait-3]: https://blog.rust-lang.org/2018/05/10/Rust-1.26.html#impl-trait

```rust,ignored
pub fn my_transform<I: Iterator>(input: I) -> impl Iterator<Item = (usize, I::Item)> {
    input.skip(3).enumerate()
}
```


<a id="c-struct-bounds"></a>
## 已经 `derive` 的数据结构不应该再使用 trait bounds 

> Data structures do not duplicate derived trait bounds (C-STRUCT-BOUNDS)


从 `derive` 属性获得 traits 时，
泛型数据结构不应该使用 trait bounds 再次获取 trait ，
也不要用另外语义的增加值。
`derive` 属性里的每个 trait 都只对实现了此 trait 的泛型参数展开成单独的 `impl` 块。

```rust,ignored
// Prefer this:
#[derive(Clone, Debug, PartialEq)]
struct Good<T> { /* ... */ }

// Over this:
#[derive(Clone, Debug, PartialEq)]
struct Bad<T: Clone + Debug + PartialEq> { /* ... */ }
```

像 `Bad` 类型那样重复实现已获得的 trait 是不必要的，
而且会导致向后不兼容。
为了理解这一点，基于前面的例子，考虑给结构体实现 `PartialOrd` trait ：

```rust,ignored
// Non-breaking change:
#[derive(Clone, Debug, PartialEq, PartialOrd)]
struct Good<T> { /* ... */ }

// Breaking change:
#[derive(Clone, Debug, PartialEq, PartialOrd)]
struct Bad<T: Clone + Debug + PartialEq + PartialOrd> { /* ... */ }
```

一般来说，给数据结构增加 trait bound 是非兼容性更改，
因为使用这个结构的人需要修改代码来满足额外的 bound 。
使用 `derive` 属性来增加标准库里的 trait 是兼容性更改。

以下 traits 绝不应该用在数据结构的 trait bounds 里：

- `Clone`
- `PartialEq`
- `PartialOrd`
- `Debug`
- `Display`
- `Default`
- `Serialize`
- `Deserialize`
- `DeserializeOwned`

对于 `derive` 不支持的 trait，
定义数据结构时需不需要使用 trait bounds 没有严格而清晰的结论。
比如 `Read` 和 `Write` trait ，它们在定义结构时 既能传达类型预期的行为，
又能限制住将来拓展新的 trait 。
而且在数据结构上使用 trait bounds 会比使用 `derive` traits 来限制泛型更简单。

---

但是 有三种情况 **必须** 使用 trait bounds 语法：

1. 数据结构的关联类型是基于 trait 的时候。
2. 限制泛型的 trait 是 `?Sized` 的时候。
3. 已经有 `Drop` impl 的数据结构，在需要用 trait 限制泛型的时候。 \
    Rust 目前要求所有数据结构上的泛型 trait bounds 都要出现在 `Drop` impl 上。[^drop-impl]

来自标准库的例子：

1. [`std::borrow::Cow`] 指向基于 `Borrow` trait 的关联类型
2. [`std::boxed::Box`] 在 `Sized` bound 之外进行操作
3. [`std::io::BufWriter`] 在 `Drop` impl 里需要 trait bound

[`std::borrow::Cow`]: https://doc.rust-lang.org/std/borrow/enum.Cow.html
[`std::boxed::Box`]: https://doc.rust-lang.org/std/boxed/struct.Box.html
[`std::io::BufWriter`]: https://doc.rust-lang.org/std/io/struct.BufWriter.html
[`std::io::BufWriter`-impl-Drop]: https://doc.rust-lang.org/src/std/io/buffered/bufwriter.rs.html#150-156

[^drop-impl]: 译者注：这句话很难理解和翻译，可能来自于 [issue 6] 。译者是观察
`std::io::BufWriter` 的 `impl Drop` [源码][`std::io::BufWriter`-impl-Drop] 
而做出这句翻译。正文原话是\
*Rust currently requires all trait bounds on the `Drop` impl are also present 
on the data structure.*

[issue 6]: https://github.com/rust-lang/api-guidelines/issues/6
