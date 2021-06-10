# 互通互用


<a id="c-common-traits"></a>
## 类型应尽早实现常见的 traits 

> Types eagerly implement common traits (C-COMMON-TRAITS)

Rust 的 trait 系统坚持 [孤儿原则][orphan] ：大致说的是，
每个 `impl` 块必须
1. 要么存在于定义 trait 的 crate 中，
2. 要么存在于给类型实现 trait 的 crate 中。

所以，定义新类型的 crates 应该尽早实现所有合适的、常见的 traits 。

为什么呢？想想下面的情况：

* `std` crate 定义了 `Display` trait 。
* `url` crate 定义了 `Url` 类型，该类型没有实现 `Display` trait 。
* `webapp` crate 导入 `std` 和 `url` crate ，

然而，没法在 `webapp` crate 中给 `Url` 增加 `Display` trait，
因为类型和 trait 都不在 `webapp` 中定义。\
（注意：[newtype] 模式可以提供一个高效但不太方便的替代办法。）

[orphan]:https://doc.rust-lang.org/book/ch10-02-traits.html?highlight=orphan#implementing-a-trait-on-a-type
[newtype]:https://doc.rust-lang.org/book/ch19-03-advanced-traits.html?highlight=orphan#using-the-newtype-pattern-to-implement-external-traits-on-external-types

`std` 中可给类型实现的、最重要的、常见的 traits 有：

- [`Copy`](https://doc.rust-lang.org/std/marker/trait.Copy.html)
- [`Clone`](https://doc.rust-lang.org/std/clone/trait.Clone.html)
- [`Eq`](https://doc.rust-lang.org/std/cmp/trait.Eq.html)
- [`PartialEq`](https://doc.rust-lang.org/std/cmp/trait.PartialEq.html)
- [`Ord`](https://doc.rust-lang.org/std/cmp/trait.Ord.html)
- [`PartialOrd`](https://doc.rust-lang.org/std/cmp/trait.PartialOrd.html)
- [`Hash`](https://doc.rust-lang.org/std/hash/trait.Hash.html)
- [`Debug`](https://doc.rust-lang.org/std/fmt/trait.Debug.html)
- [`Display`](https://doc.rust-lang.org/std/fmt/trait.Display.html)
- [`Default`](https://doc.rust-lang.org/std/default/trait.Default.html)

给类型实现 `Default` trait 和空的 `new` 构造函数是常见和有必要的。\
`new` 是 Rust 中常规的构造函数，所以不使用参数来构造基本的类型时，
`new` 对使用者来说就理应存在。\
`default` 方法功能上与 `new` 方法一致，所以也应当存在。


<a id="c-conv-traits"></a>
## 使用 `From`, `AsRef`, `AsMut` trait 来转换类型  

> Conversions use the standard traits `From`, `AsRef`, `AsMut` (C-CONV-TRAITS)

以下转换类型的 traits 在合理的时候 **应该** 被实现：

- [`From`](https://doc.rust-lang.org/std/convert/trait.From.html)
- [`TryFrom`](https://doc.rust-lang.org/std/convert/trait.TryFrom.html)
- [`AsRef`](https://doc.rust-lang.org/std/convert/trait.AsRef.html)
- [`AsMut`](https://doc.rust-lang.org/std/convert/trait.AsMut.html)

以下转换类型的 traits 在任何时候都 **不应该** 被实现：

- [`Into`](https://doc.rust-lang.org/std/convert/trait.Into.html)
- [`TryInto`](https://doc.rust-lang.org/std/convert/trait.TryInto.html)

因为这两个 traits 基于 `From` 、 `TryFrom` trait 进行覆盖实现 (blanket impl) ，
所以只需要实现 `From` 、 `TryFrom` 。

来自标准库的例子：

- `u32` 实现了 `From<u16>` ，因为更小范围的整数总是可以转化为更大范围的整数。
- `u16` 没有实现 `From<u32>` ，因为如果超过 `u16` 范围的整数不可能转化为 `u16` 。
- `u16` 实现了 `TryFrom<u32>` ，如果整数大于 `u16` 的范围，那么返回错误。
- [`IpAddr`] 实现了 [`From<Ipv6Addr>`] ，[`IpAddr`] 能够以 v4 和 v6 两种 IP 地址表示。

[`From<Ipv6Addr>`]: https://doc.rust-lang.org/std/net/struct.Ipv6Addr.html
[`IpAddr`]: https://doc.rust-lang.org/std/net/enum.IpAddr.html


<a id="c-collect"></a>
## 给集合实现 `FromIterator` 和 `Extend` trait

> Collections implement `FromIterator` and `Extend` (C-COLLECT)

[`FromIterator`] 和 [`Extend`] trait 让集合 (collections) 方便地使用以下迭代器方法：

[`FromIterator`]: https://doc.rust-lang.org/std/iter/trait.FromIterator.html
[`Extend`]: https://doc.rust-lang.org/std/iter/trait.Extend.html

- [`Iterator::collect`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.collect)
- [`Iterator::partition`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.partition)
- [`Iterator::unzip`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.unzip)

`FromIterator` 从迭代器中生成新的集合，
`Extend` 从迭代器中追加元素到已存在的集合。

来自标准库的例子：

- [`Vec<T>`] 实现了 `FromIterator<T>` 和 `Extend<T>` 。

[`Vec<T>`]: https://doc.rust-lang.org/std/vec/struct.Vec.html


<a id="c-serde"></a>
## 给数据结构实现 Serde 的 `Serialize` 和 `Deserialize` trait 

> Data structures implement Serde's `Serialize`, `Deserialize` (C-SERDE)

具有数据结构功能的类型应该实现 `Serialize` 和 `Deserialize` trait 。

[`Serialize`]: https://docs.serde.rs/serde/trait.Serialize.html
[`Deserialize`]: https://docs.serde.rs/serde/trait.Deserialize.html

对于数据结构 (data structure) ，界定显然是和显然不是 **之间** 的类型是很困难的。\

这两个是界限分明的例子：\
[`LinkedHashMap`] 和 [`IpAddr`] 是数据结构，
从 JSON 中读取 `LinkedHashMap` / `IpAddr` ，
或者通过 IPC 发送其中一种数据到另一个进程 都十分合理。\
[`LittleEndian`] 不是数据结构，它在 `byteorder` crate 中被用来编译期优化字节，
尤其是字节的顺序，而且实际上 `LittleEndian` 永远不存在于运行时。

而 #rust 和 #serde IRC 通信就是很让这种界限模糊的情况得到良好的处理。
如果某个 crate 因为某些原因还没有把 Serde 作为依赖，那么它可能希望把 Serde 相关的
impls 块放在 Cargo cfg 后面用作条件编译。从而下游的库需要这些 Serde impls 的时候，
就编译 Serde ，不需要的时候不编译 Serde 。

为了和其他基于 Serde 的库一致，Cargo cfg 的名称应该是简单的 `"serde"` ，
而不要使用 `"serde_impls"` 或  `"serde_serialization"` 当作名称。

[`LinkedHashMap`]: https://docs.rs/linked-hash-map/0.4.2/linked_hash_map/struct.LinkedHashMap.html
[`IpAddr`]: https://doc.rust-lang.org/std/net/enum.IpAddr.html
[`LittleEndian`]: https://docs.rs/byteorder/1.0.0/byteorder/enum.LittleEndian.html

不使用 derive 参数的话，通常实现方式如下：

```toml
[dependencies]
serde = { version = "1.0", optional = true }
```

```rust,ignored
#[cfg(feature = "serde")]
extern crate serde;

struct T { /* ... */ }

#[cfg(feature = "serde")]
impl Serialize for T { /* ... */ }

#[cfg(feature = "serde")]
impl<'de> Deserialize<'de> for T { /* ... */ }
```

使用 derive 参数的话，实现方式为：

```toml
[dependencies]
serde = { version = "1.0", optional = true, features = ["derive"] }
```

```rust,ignored
#[cfg(feature = "serde")]
#[macro_use]
extern crate serde;

#[cfg_attr(feature = "serde", derive(Serialize, Deserialize))]
struct T { /* ... */ }
```


<a id="c-send-sync"></a>
## 类型应尽可能实现 `Send` 和 `Sync` trait 

> Types are `Send` and `Sync` where possible (C-SEND-SYNC)

在编译器确认类型适合实现 `Send` 和 `Sync` trait 的时候，它们是自动实现的。

[`Send`]: https://doc.rust-lang.org/std/marker/trait.Send.html
[`Sync`]: https://doc.rust-lang.org/std/marker/trait.Sync.html

对于操纵裸指针 (raw pointers) 的类型，实现这两个 trait 就要十分谨慎，
因为它们表明是线程安全的。\
像下面的单元测试有助于察觉到是否因 `Send` 或 `Sync` 导致无意间的性能倒退。

```rust,ignored
#[test]
fn test_send() {
    fn assert_send<T: Send>() {}
    assert_send::<MyStrangeType>();
}

#[test]
fn test_sync() {
    fn assert_sync<T: Sync>() {}
    assert_sync::<MyStrangeType>();
}
```


<a id="c-good-err"></a>
## Error 类型 十分直观和有用 

> Error types are meaningful and well-behaved (C-GOOD-ERR)

Error 类型 是 `Result<T, E>` 中的 `E` 类型，在你的 crate 中
`Result<T, E>` 是被公共 (public) 函数返回的。\
Error 类型 总是应该实现 [`std::error::Error`] trait ：\
在错误处理库中用这个 trait 来对不同类型的错误进行抽象（比如 [`error-chain`] crate ）；\
也可以使用这个 trait 的 [`.source()`] 方法来得到更底层的错误。 

[`std::error::Error`]: https://doc.rust-lang.org/std/error/trait.Error.html
[`error-chain`]: https://docs.rs/error-chain
[`source()`]: https://doc.rust-lang.org/std/error/trait.Error.html#method.source

此外，Error 类型 应该实现 [`Send`] 和 [`Sync`] trait 。\
不具备 [`Send`] 的 Error 类型 不能被 [`thread::spawn`] 运行的线程返回；\
不具备 [`Sync`] 的 Error 类型 不能在不同的线程间使用 [`Arc`] 来传递。\
这些在多线程应用软件中处理基础错误时是常见且必须的。

[`Send`]: https://doc.rust-lang.org/std/marker/trait.Send.html
[`Sync`]: https://doc.rust-lang.org/std/marker/trait.Sync.html
[`thread::spawn`]: https://doc.rust-lang.org/std/thread/fn.spawn.html
[`Arc`]: https://doc.rust-lang.org/std/sync/struct.Arc.html

`Send` 和 `Sync` 在使用 [`std::io::Error::new`] 进行自定义 IO 错误时扮演了重要角色，
因为这里传入的 Error 类型 需要实现 `Error + Send + Sync` trait 。

[`std::io::Error::new`]: https://doc.rust-lang.org/std/io/struct.Error.html#method.new

在返回 Error trait 对象的函数里，需要对这条原则保持审慎。
比如 [`reqwest::Error::get_ref`] 函数。
通常 `Error + Send + Sync + 'static` 组合适合对大多数调用场景。
附加的 `'static` 能利用 [`Error::downcast_ref`] 方法来获取 Error trait 对象 。


[`reqwest::Error::get_ref`]: https://docs.rs/reqwest/0.7.2/reqwest/struct.Error.html#method.get_ref
[`Error::downcast_ref`]: https://doc.rust-lang.org/std/error/trait.Error.html#method.downcast_ref-2

不要把 `()` 作为 Error 类型 ，即便对于一个不带有帮助性的额外信息的错误，
也不要让这个错误是 `()` 类型 。

- `()` 没有实现 `Error` ，所以它无法和 `error-chain` 这样的错误处理库一起使用。
- `()` 没有实现 `Display` ，所以使用者无法打印自己写的错误信息。
- `()` 实现了 `Debug` ，但是当使用者 `unwrap` 错误的时候，它没提供任何有用的东西。
- 对于实现了 `From<()>` 的 Error 类型的下游库 ，`()` 在语义上是毫无意义的，
  从而无法使用 `?` 操作符。

需要做的是，针对你的 crate 或者单个函数来定义有实际意义的 Error 类型 。
给合适的 Error 类型 实现 `Error` 和 `Display` trait 。
对于无实际意义的错误，利用 unit 结构体来实现 `Error` 和 `Display` trait 。

```rust,ignored
use std::error::Error;
use std::fmt::Display;

// Instead of this...
fn do_the_thing() -> Result<Wow, ()>

// Prefer this...
fn do_the_thing() -> Result<Wow, DoError>

#[derive(Debug)]
struct DoError;

impl Display for DoError { /* ... */ }
impl Error for DoError { /* ... */ }
```

Error 类型 实现的 `Display` trait 通常应该提供简明的错误信息 (error messages)，
而且信息用全小写，末尾不带标点符号。\
不应该实现 [`Error::description()`] trait ，因为它已经被弃用。
而应该实现 `Display` trait 来打印错误[^Display]。

[`Error::description()`]: https://doc.rust-lang.org/std/error/trait.Error.html#tymethod.description

来自标准库的例子：

- [`ParseBoolError`] ：从字符串解析 bool 值时返回这个错误。

[`ParseBoolError`]: https://doc.rust-lang.org/std/str/struct.ParseBoolError.html

一些错误信息的例子：

- "unexpected end of file" 
- "provided string was not \`true\` or \`false\`"
- "invalid IP address syntax"
- "second time provided was later than self"
- "invalid UTF-8 sequence of {} bytes from index {}"
- "environment variable was not valid unicode: {:?}"


[^Display]: 译者注：
[`ToString`](https://doc.rust-lang.org/std/string/trait.ToString.html#tymethod.to_string)
给所有具有 `Display` trait 的类型自动实现了 `.to_string()` 方法，从而获得错误信息的字符串。
<a id="c-num-fmt"></a>

## 二进制数类型应提供 `Hex`, `Octal`, `Binary` 的格式化方式 

> Binary number types provide `Hex`, `Octal`, `Binary` formatting (C-NUM-FMT)

- [`std::fmt::UpperHex`](https://doc.rust-lang.org/std/fmt/trait.UpperHex.html)
- [`std::fmt::LowerHex`](https://doc.rust-lang.org/std/fmt/trait.LowerHex.html)
- [`std::fmt::Octal`](https://doc.rust-lang.org/std/fmt/trait.Octal.html)
- [`std::fmt::Binary`](https://doc.rust-lang.org/std/fmt/trait.Binary.html)

这些 traits 在 `{:X}`, `{:x}`, `{:o}`, `{:b}` 格式化分类符 (format specifiers)
下用来控制类型的呈现方式。

对你认为会进行 位操作（比如 `|` 、  `&`）的任何数字类型实现这些 traits ——
尤其适合 位标志 (bitflag) 类型。
像 `struct Nanoseconds(u64)` 这样的数值单位类型就可能无需实现这些 trait 。

<a id="c-rw-value"></a>
## reader/writer 泛型函数使用 `R: Read` 和 `W: Write` 参数传值 

> Generic reader/writer functions take `R: Read` and `W: Write` by value (C-RW-VALUE)

标准库里有以下两个 impl 块：

```rust,ignored
impl<'a, R: Read + ?Sized> Read for &'a mut R { /* ... */ }

impl<'a, W: Write + ?Sized> Write for &'a mut W { /* ... */ }
```

这意味着接收 `R: Read` 或者 `W: Write` 泛型参数值的 所有函数
都可以在需要的时候传入 可变引用 (mut reference) 。

在这些函数的文档里，需要简要地提醒使用者可以传入可变引用。
Rust 新手经常对此感到疑惑。
已经打开了文件，并且想从文件中读取数据的时候，新手往往卡在这一步。
解决的办法是，根据上面两种情况的一种，传入 `&mut f` ，
而不是传入 `f` 作为参数。

例子：

- [`flate2::read::GzDecoder::new`]
- [`flate2::write::GzEncoder::new`]
- [`serde_json::from_reader`]
- [`serde_json::to_writer`]

[`flate2::read::GzDecoder::new`]: https://docs.rs/flate2/1.0.20/flate2/read/struct.GzDecoder.html#method.new
[`flate2::write::GzEncoder::new`]: https://docs.rs/flate2/1.0.20/flate2/write/struct.GzEncoder.html#method.new
[`serde_json::from_reader`]: https://docs.serde.rs/serde_json/fn.from_reader.html
[`serde_json::to_writer`]: https://docs.serde.rs/serde_json/fn.to_writer.html
