# 文档编写


<a id="c-crate-doc"></a>
## crate 级别的文档应该详实有例 

> Crate level docs are thorough and include examples (C-CRATE-DOC)

参考 [RFC 1687] 。

[RFC 1687]: https://github.com/rust-lang/rfcs/pull/1687


<a id="c-example"></a>
## 每个条目都应该有例子 

> All items have a rustdoc example (C-EXAMPLE)

每个公有的 模块、 trait 、结构体、枚举体、函数、方法、宏、类型定义 ，
都应该有展示其功能和使用方法的例子。

这条原则应该在合理的地方使用。

以链接的形式关联另一个条目的使用例子可能就足够了。
比如一个函数用到一个具体的类型，合适的做法是：
给这个函数或者类型写一个例子，然后链接这个例子到另一个。

写例子的目的不一定仅仅是展示 **如何使用** 这个条目。
阅读它的人可以理解怎样调用函数、匹配枚举体 和 其他基础的用法。
但更多时候，例子通常用来展示 **为什么** 使用者愿意去使用这个条目。

```rust,ignored
// 这就是一个关于 `.clone()` 的不好的例子。
// 它死板地展示如何调用 clone() ，而完全没有表明 *为什么* 需要这样做。
fn main() {
    let hello = "hello";

    hello.clone();
}
```


<a id="c-question-mark"></a>
## 例子应该使用 `?` 而不使用 `try!` 或者 `unwrap` 

> Examples use `?`, not `try!`, not `unwrap` (C-QUESTION-MARK)

不管你喜不喜欢这样做，例子里的代码通常会被使用者一字不落地复制下来。
使用者应该慎重地决定怎样处理每个错误。

一个容易出错的代码示例通常是按照下面的方式来写：
由 `cargo test` 编译测试代码块，

```
/// ```rust
/// # use std::error::Error;
/// #
/// # fn main() -> Result<(), Box<dyn Error>> {
/// your;
/// example?;
/// code;
/// #
/// # Ok(())
/// # }
/// ```
```

但以 `#` 开头的每行代码不会出现在读者可见的 rustdoc 里。

```rust,ignored
# use std::error::Error;
#
# fn main() -> Result<(), Box<dyn Error>> {
your;
example?;
code;
#
# Ok(())
# }
```



<a id="c-failure"></a>
## 函数涉及错误、 panic、安全性时 应该加以说明 

> Function docs include error, panic, and safety considerations (C-FAILURE)

引发错误的条件应该在 "Errors" 标题下说明。
这也适用于 trait 方法 —— 可以或者可能返回错误的方法都应该在 "Errors" 小节进行说明。

比如以下标准库里的例子，
[`std::io::Read::read`] trait 里某个方法可能会返回一个错误。

[`std::io::Read::read`]: https://doc.rust-lang.org/std/io/trait.Read.html#tymethod.read

```
/// Pull some bytes from this source into the specified buffer, returning
/// how many bytes were read.
///
/// ... lots more info ...
///
/// # Errors
///
/// If this function encounters any form of I/O or other error, an error
/// variant will be returned. If an error is returned then it must be
/// guaranteed that no bytes were read.
```

引发 panic 的条件应该在 "Panics" 标题下说明。
这也适用于 trait 方法 —— 可以或者可能导致 panic 的方法都应该在 "Panics" 小节进行说明。

标准库里的 [`Vec::insert`] 方法可能会 panic ：

[`Vec::insert`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.insert

```
/// Inserts an element at position `index` within the vector, shifting all
/// elements after it to the right.
///
/// # Panics
///
/// Panics if `index` is out of bounds.
```

没必要把所有可能想到的 panic 情况都进行说明，
尤其是如果 panic 发生在调用期间有逻辑错误的时候。
以如下的方式说明 `Display` 会 panic 就显得多余了。

```rust,ignored
/// # Panics
///
/// This function panics if `T`'s implementation of `Display` panics.
pub fn print<T: Display>(t: T) {
    println!("{}", t.to_string());
}
```

unsafe 的函数应该放在 "Safety" 小节，用来解释如何正确使用这个函数。

例如，不安全的 [`std::ptr::read`] 方法必须在以下情况才能被调用：

[`std::ptr::read`]: https://doc.rust-lang.org/std/ptr/fn.read.html

```
/// Reads the value from `src` without moving it. This leaves the
/// memory in `src` unchanged.
///
/// # Safety
///
/// Beyond accepting a raw pointer, this is unsafe because it semantically
/// moves the value out of `src` without preventing further usage of `src`.
/// If `T` is not `Copy`, then care must be taken to ensure that the value at
/// `src` is not used before the data is overwritten again (e.g. with `write`,
/// `zero_memory`, or `copy_memory`). Note that `*src = foo` counts as a use
/// because it will attempt to drop the value previously at `*src`.
///
/// The pointer must be aligned; use `read_unaligned` if that is not the case.
```


<a id="c-link"></a>
## 给相关的内容添加超链接 

> Prose contains hyperlinks to relevant things (C-LINK)

一般使用 markdown 超链接语法 `[text](url)` 来设置超链接。
链接到某个类型则可以使用 ``[`text`]`` 语法来标注，
在 docstring 底部另起一行，使用 ``[`text`]: <target>`` 来添加链接地址，
下面谈谈这里的 `<target>` 。

通常这样做来链接到这个类型的方法：

```md
[`serialize_struct`]: #method.serialize_struct
```

通常这样做来链接到其他类型：

```md
[`Deserialize`]: trait.Deserialize.html
```

也可以链接到父模块或者子模块：

```md
[`Value`]: ../enum.Value.html
[`DeserializeOwned`]: de/trait.DeserializeOwned.html
```

这条原则由 RFC 1574 正式推荐，可参考其 ["Link all the things"] 部分。

["Link all the things"]: https://github.com/rust-lang/rfcs/blob/master/text/1574-more-api-documentation-conventions.md#link-all-the-things


<a id="c-metadata"></a>
## Cargo.toml 应包含所有常见的配置数据 

> Cargo.toml includes all common metadata (C-METADATA)

在 `Cargo.toml` 的 `[package]` 部分应该要有以下内容：

- `authors`
- `description`
- `license`
- `repository`
- `readme`
- `keywords`
- `categories`

此外，这两个配置字段可选填：

- `documentation`
- `homepage`

*crates.io* 默认会把已发布的 crate 文档链接到 [*docs.rs*] 。
`documentation` 配置信息在文档发布到 *docs.rs* 之外的地方时才需要填写。
比如这个 crate 链接了不在 *docs.rs* 上构建的共享库。

[*docs.rs*]: https://docs.rs

`homepage` 配置信息只填写除源码仓库或 API 文档网址之外的单独网站。
不要让 `homepage` 和 `documentation` 或者 `repository` 的值重复。
比如 serde 设置其 `homepage` 为 *https://serde.rs* 。

[C-HTML-ROOT]: #c-html-root
<a id="c-html-root"></a>
## 设置 html_root_url 属性 "https://docs.rs/CRATE/X.Y.Z"  

> Crate sets html_root_url attribute (C-HTML-ROOT)

<!--
Remove this guideline once rustdoc links no-deps documentation with no
html_root_url to docs.rs by default.
https://github.com/rust-lang/rust/issues/42301
-->

加入 crate 使用 docs.rs 作为其主要的 API 文档平台，
那么 `html_root_url` 应该指向 `"https://docs.rs/CRATE/MAJOR.MINOR.PATCH"` 地址。[^html_root_url]

在编译下游 crates 时， `html_root_url` 属性告诉 rustdoc 
如何生成指向条目的 URLs 。
如果没有这个属性，依赖于这个 crate 的其他 crates 里的文档链接地址就不正确。

```rust,ignored
#![doc(html_root_url = "https://docs.rs/log/0.3.8")]
```

因为这个 URL 包含确定的版本号，所以这个版本号必须和 `Cargo.toml` 里的版本号同步。
[`version-sync`] crate 能帮助你做集成测试，当 `html_root_url` 
里的版本号落后于 crate 的版本号时，测试不通过。

[`version-sync`]: https://crates.io/crates/version-sync

如果你不喜欢使用 [`version-sync`] 提供的测试机制，那么建议你在 `Cargo.toml` 
里 version 一行添加注释来 *提醒你* 保持这两者同步，就像：

```toml
version = "0.3.8" # remember to update html_root_url
```

对于发布在 docs.rs 之外的文档， `html_root_url` 得设置成一个增加
*crate 名 + index.html* 之后能访问到 crate 根目录文档的地址。
比如 crate 的根目录文档地址是 `"https://api.rocket.rs/rocket/index.html"` ，
那么 `html_root_url` 的值应该填成 `"https://api.rocket.rs"` 。

[^html_root_url]: 译者注：当 rustdoc 支持不设置 `html_root_url` 默认就指向 docs.rs
的功能之后，这条原则就可以删除。见 
[issue 42301](https://github.com/rust-lang/rust/issues/42301) 。
此外，你还可以看看文档 [rustdoc: html_root_url] 和 [cargo doc] 。

[rustdoc: html_root_url]:https://doc.rust-lang.org/rustdoc/the-doc-attribute.html#html_root_url
[cargo doc]: https://doc.rust-lang.org/cargo/commands/cargo-doc.html

<a id="c-relnotes"></a>
## 发布时 记录该版本的重大变化 

> Release notes document all significant changes (C-RELNOTES)

crate 的使用者能从版本说明 (release note) 中找到每个发布版本的更改概要。
这些版本说明或者其链接应该在 crate 根文档 以及/或者 
Cargo.toml 填写的仓库说明 里有介绍。

不兼容的变更 （正如 [RFC 1105] 所定义） 应该清楚地写在版本说明里。

如果使用 Git 来追踪 crate 源代码，那么每个发布到 *crates.io* 上的版本
应该有对应的标签 (tag) 来标记这次已发布的提交记录。
不使用 Git 的版本控制工具也应该用类似的方式进行处理。
参考以下 Git 命令：

```bash
# Tag the current commit
GIT_COMMITTER_DATE=$(git log -n1 --pretty=%aD) git tag -a -m "Release 0.3.0" 0.3.0
git push --tags
```

有注释的 tag 会更好，因为一些 Git 命令会在有注释 tag 存在的情况下
忽略不带注释的 tag 。

[RFC 1105]: https://github.com/rust-lang/rfcs/blob/master/text/1105-api-evolution.md

例子：

- [Serde 1.0.0 release notes](https://github.com/serde-rs/serde/releases/tag/v1.0.0)
- [Serde 0.9.8 release notes](https://github.com/serde-rs/serde/releases/tag/v0.9.8)
- [Serde 0.9.0 release notes](https://github.com/serde-rs/serde/releases/tag/v0.9.0)
- [Diesel change log](https://github.com/diesel-rs/diesel/blob/master/CHANGELOG.md)


<a id="c-hidden"></a>
## 文档不应该展示无太大帮助的实现细节 

> Rustdoc does not show unhelpful implementation details (C-HIDDEN)

rustdoc 应该包含帮助使用者充分使用该 crate 的所有信息。
解释相关的实现细节是可以的，
但是不应该在文档里展开论述这些细节。

尤其应该挑选哪些条目可以在 rustdoc 展示出来 —— 
展示那些让使用者完全掌握使用这个 crate 的内容，
其他的内容不展示。
下面代码中给 `PublicError` 实现 `From<PrivateError>` 的 rustdoc 文档会默认展示出来，
使用 `#[doc(hidden)]` 来隐藏它，因为用户不会在代码里面用到私有的 `PrivateError` ，
所以这个 impl 块对用户来说完全无关。

```rust,ignored
// This error type is returned to users.
pub struct PublicError { /* ... */ }

// This error type is returned by some private helper functions.
struct PrivateError { /* ... */ }

// Enable use of `?` operator.
#[doc(hidden)]
impl From<PrivateError> for PublicError {
    fn from(err: PrivateError) -> PublicError {
        /* ... */
    }
}
```

[`pub(crate)`] 是一个很棒的工具，它从公有 API 移除掉实现的细节：\
让条目于 定义所在的模块之外 被使用，但 定义所在的 crate 之外 无法被使用。
（仅让条目在 crate 内可见，在 crate 之外不可见）

[`pub(crate)`]: https://github.com/rust-lang/rfcs/blob/master/text/1422-pub-restricted.md
