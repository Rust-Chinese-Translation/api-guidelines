# 必要项


<a id="c-stable"></a>
## 稳定版 crate 必须具有稳定的公有依赖 

> Public dependencies of a stable crate are stable (C-STABLE)

一个稳定版 (stable) 的 crate （版本号>=1.0.0） ，其所有的公有依赖都必须是稳定的。

公有依赖 (public dependencies) 指当前 crate 使用的公有 API 所来自的 crate 。

```rust,ignored
pub fn do_my_thing(arg: other_crate::TheirThing) { /* ... */ }
```

对于一个 crate 定义这样的函数，如果 `other_crate` 不是稳定的话，
那么这个 crate 就不是稳定的。

要小心，公有依赖可能在不经意之间出现在意料之外的地方。

```rust,ignored
pub struct Error {
    private: ErrorImpl,
}

enum ErrorImpl {
    Io(io::Error),
    // 这没问题，即使 `other_crate` 不是稳定的，因为 `ErrorImpl` 是私有的
    Dep(other_crate::Error),
}

// 不！这让可能不稳定的 `other_crate` 暴露在当前 crate 的公共 API 里
impl From<other_crate::Error> for Error {
    fn from(err: other_crate::Error) -> Self {
        Error { private: ErrorImpl::Dep(err) }
    }
}
```


<a id="c-permissive"></a>
## crate 及其依赖必须有许可证 

> Crate and its dependencies have a permissive license (C-PERMISSIVE)

Rust 项目组编写的软件都是 [MIT] 和 [Apache 2.0] 双重许可的。
如果 crates 需要最大程度地与 Rust 生态兼容，那么建议也这么做。
下面谈谈选择其他的许可。

这里不会详细解释 Rust 版权， [Rust FAQ] 谈到了一小部分。
这些原则涉及到 Rust 能否被通用的问题，
在选择许可方面说得不多。

[MIT]: https://github.com/rust-lang/rust/blob/master/LICENSE-MIT
[Apache 2.0]: https://github.com/rust-lang/rust/blob/master/LICENSE-APACHE
[Rust FAQ]: https://github.com/dtolnay/rust-faq#why-a-dual-mitasl2-license

在 `Cargo.toml` 文件的 `license` 字段填上你的项目适用的许可证。

```toml
[package]
name = "..."
version = "..."
authors = ["..."]
license = "MIT OR Apache-2.0"
```

在仓库的根目录添加 `LICENSE-APACHE` 和 `LICENSE-MIT` 文件，
文件里面涵盖许可的正文。
你可以在 choosealicense.com 获得这些许可，
比如 [Apache-2.0](https://choosealicense.com/licenses/apache-2.0/)
和 [MIT](https://choosealicense.com/licenses/mit/) 。

然后在 README.md 文件的最后写上：

```
## License

Licensed under either of

 * Apache License, Version 2.0
   ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
 * MIT license
   ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.

## Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be
dual licensed as above, without any additional terms or conditions.
```

除了 MIT/Apache-2.0 双重许可之外，
另一种 Rust crate 作者们常用的许可声明方式是使用单独的 MIT 或者 BSD 许可。
BSD 许可方式完全与 Rust 的 MIT 许可方式兼容，
因为它对 MIT 许可做了最小程度的限制。

不建议 想要完全兼容 Rust 许可的 crates 仅仅使用 Apache 许可证。
虽然 Apache 许可证比 MIT 和 BSD 更严格，
但是在某些场景下会打击或者打消使用这些 cates 的念头，
所以只使用 Apache 许可证 的软件 在大多数 Rust 技术栈能使用的地方 无法被使用。

一个 crate 依赖的许可证可能影响和限制 crate 自身的分发，
所以 自由许可 的 crate 通常应仅依赖 自由许可 的 crates 。
