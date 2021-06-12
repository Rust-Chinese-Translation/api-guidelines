# 可依赖


<a id="c-validate"></a>
## 函数会验证其参数 

> Functions validate their arguments (C-VALIDATE)

Rust APIs 不遵循 [稳健性法则][robustness principle]：
对己方保守，对他方宽容。

[robustness principle]: http://en.wikipedia.org/wiki/Robustness_principle

正相反， Rust 代码应该在任何可以 **强制** 校验其输入的时候进行强制校验。

可以通过以下机制来实现强制验证。出现的顺序代表性能高低。

### 静态验证

把输入限制在类型上，排除掉不符合的输入。
比如，请这样写：


```rust,ignored
fn foo(a: Ascii) { /* ... */ }
```

而不是这样写：

```rust,ignored
fn foo(a: u8) { /* ... */ }
```

`Ascii` 是对 `u8` 类型的包装器 (wrapper) ，从而保证最高位是零。
参考 [newtype][C-NEWTYPE] 模式，来详细了解生成类型安全的包装器。

静态验证 (static enforcement) 通常具有很少的运行时代价：
它的代价在于边界，例如在 `u8` 首先转换为 `Ascii` 类型的时候。
它也能在早期发现 bugs ，在编译期发现，而不是在运行失败的时候才发现。

可是，有些性质很难或者不可能用类型来描述。

[C-NEWTYPE]: type-safety.html#c-newtype

### 动态验证 

动态验证 (dynamic enforcement) 即 在处理的时候（如果有必要的话处理之前） 校验输入。
动态检查通常比静态检查更容易实现，但是有以下几个缺点：

- 运行时开销（除非动态检查是处理输入的一部分）
- bugs 检测不及时
- 带来失败的情况，用户的代码需要通过 `panic!` ，或者 `Result` / `Option` 
  类型处理。

1. 使用 [`debug_assert!`] 。
在构建生产版本的时候可以关掉高花销的检查。

2. 增加不进行检查的同类函数。
通常用 `_unchecked` 后缀来标记这个函数不进行某些检查，
或者把函数放在 `raw` 的子模块下面。\
在下面两个场景下使用 unchecked 函数是明智的：
为了更高的性能；使用者确信输入的数据是有效的。

[`debug_assert!`]: http://129.28.186.100/rust-docs/rust/html/std/macro.debug_assert.html

<a id="c-dtor-fail"></a>
## 析构函数不该运行失败 

> Destructors never fail (C-DTOR-FAIL)

析构函数 (destructor) 在某项任务失败的时候也会执行，
如果析构函数运行失败的话程序就会中断且退出。

所以为了不让析构函数失败，应该单独提供用于检查的方法。
比如 `close` 方法就返回 `Result` 来表明遇到了问题。


<a id="c-dtor-block"></a>
## 阻塞的析构函数应有替代的办法 

> Destructors that may block have alternatives (C-DTOR-BLOCK)

同样，析构函数不应该调用造成阻塞的操作，
阻塞会让调试更困难。

重申一遍，考虑为不会失败且非阻塞的析构提供单独的方法。

[destructor]: https://doc.rust-lang.org/nightly/reference/destructors.html
