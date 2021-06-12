# 宏


<a id="c-evocative"></a>
## 输入语法与输出语法一致

> Input syntax is evocative of the output (C-EVOCATIVE)

Rust 宏系统允许你创造出你想要的输入语法。
尽量让输入的语法是让大多数人感到熟悉的，
仿照现存的 Rust 语法以便让使用者的代码风格统一。
注意对关键字和标点符号的选择与更改。

好的标准是：宏输入的语法与输出的结果不会相差太大，尤其在关键字和标点符号方面。

比如你写的宏要根据输入来声明一个具体的结构体，那么就以 `struct` 关键字开头，
后接结构体的名称。

```rust,ignored
// 这种方式更好
bitflags! {
    struct S: u32 { /* ... */ }
}

// 这种方式就不太好
bitflags! {
    S: u32 { /* ... */ }
}

// 这种方式也不太好
bitflags! {
    flags S: u32 { /* ... */ }
}
```

另一个例子是分号和逗号之间如何做出选择。
Rust 的常量 (constants) 以分号作为结尾，所以如果你写的宏要定义一系列常量，
那么应该在输入语法里书写分号，即使语法看起来与 Rust 的语法有些不同。

```rust,ignored
// 原始的常量定义语法后面接的是分号
const A: u32 = 0b000001;
const B: u32 = 0b000010;

// 所以这种方式更好
bitflags! {
    struct S: u32 {
        const C = 0b000100;
        const D = 0b001000;
    }
}

// 这种方式不太好
bitflags! {
    struct S: u32 {
        const E = 0b010000,
        const F = 0b100000,
    }
}
```

宏是多样化的，以致于这里举的例子不会适合所有场景。
但是的确要仔细思考如何应用同一套规则。


<a id="c-macro-attr"></a>
## 宏与属性形成有机的整体 

> Item macros compose well with attributes (C-MACRO-ATTR)

生成超过一个条目 ([item]) 的宏，应该支持对每个条目添加属性。
一个常见的例子是，把单独的条目放在 cfg 后面：

[item]:https://doc.rust-lang.org/nightly/reference/items.html

```rust,ignored
bitflags! {
    struct Flags: u8 {
        #[cfg(windows)]
        const ControlCenter = 0b001;
        #[cfg(unix)]
        const Terminal = 0b010;
    }
}
```

生成结构体或枚举体的宏，应该支持添加宏属性，
方便让输出结果使用 derive 属性。

```rust,ignored
bitflags! {
    #[derive(Default, Serialize)]
    struct Flags: u8 {
        const ControlCenter = 0b001;
        const Terminal = 0b010;
    }
}
```


<a id="c-anywhere"></a>
## 生成条目的宏可以在条目被允许的地方使用 

> Item macros work anywhere that items are allowed (C-ANYWHERE)

Rust 允许条目 ([item]) 有模块级别那样的作用域或者出现在类似函数这样小的作用域里。
生成条目的宏应该和普通的条目一样，在这些地方可以正常使用。
测试这些宏的时，应该至少在模块和函数作用域里都调用宏。

```rust,ignored
#[cfg(test)]
mod tests {
    test_your_macro_in_a!(module);

    #[test]
    fn anywhere() {
        test_your_macro_in_a!(function);
    }
}
```

下面这个例子展示了错误的宏：
这个宏能在模块作用域里运行，但不能在函数作用域里运行。

```rust,ignored
macro_rules! broken {
    ($m:ident :: $t:ident) => {
        pub struct $t;
        pub mod $m {
            pub use super::$t;
        }
    }
}

broken!(m::T); // 运行通过，可以展开成 T 和 m::T

fn g() {
    broken!(m::U); // 编译失败，super::U 指的是上级模块，而不是函数 g
}
```


<a id="c-macro-vis"></a>
## 生成条目的宏应支持可见性分类符 

> Item macros support visibility specifiers (C-MACRO-VIS)

宏应遵循 Rust 对条目可见性 ([visibility]) 的语法要求：
默认是私有的，如果使用 `pub` 则表明条目是公有的。

[visibility]:https://doc.rust-lang.org/nightly/reference/visibility-and-privacy.html

```rust,ignored
bitflags! {
    struct PrivateFlags: u8 {
        const A = 0b0001;
        const B = 0b0010;
    }
}

bitflags! {
    pub struct PublicFlags: u8 {
        const C = 0b0100;
        const D = 0b1000;
    }
}
```


<a id="c-macro-ty"></a>
## 类型分类符 `$t:ty` 是灵活的 

> Type fragments are flexible (C-MACRO-TY)

如果你写的宏接收 [`$t:ty`] 类型 [分类符][fragment-specifier] 作为输入，
那么输入的内容应该与以下代码一起使用：

- 原生类型： `u8`, `&str`
- 相对路径： `m::Data`
- 绝对路径： `::base::Data`
- 上级相对路径： `super::Data`
- 泛型： `Vec<String>`

下面这个例子展示了错误的宏：
这个宏能在很好地与原生类型、绝对路径一起使用，
但不能和相对路径一起使用。

```rust
macro_rules! broken {
    ($m:ident => $t:ty) => {
        pub mod $m {
            pub struct Wrapper($t);
        }
    }
}

broken!(a => u8); // okay

broken!(b => ::std::marker::PhantomData<()>); // okay

struct S;
broken!(c => S); // 编译失败： `S` not found in this scope
```

[`$t:ty`]:https://doc.rust-lang.org/nightly/reference/types.html
[fragment-specifier]:https://doc.rust-lang.org/nightly/reference/macros-by-example.html#metavariables
