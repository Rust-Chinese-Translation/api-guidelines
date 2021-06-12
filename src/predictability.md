# 可预测


<a id="c-smart-ptr"></a>
## 智能指针不增加固有方法 

> Smart pointers do not add inherent methods (C-SMART-PTR)

举例来说，以下代码说明了为什么 [`Box::into_raw`] 是这样定义的：

[`Box::into_raw`]: https://doc.rust-lang.org/std/boxed/struct.Box.html#method.into_raw

```rust,ignored
impl<T> Box<T> where T: ?Sized {
    fn into_raw(b: Box<T>) -> *mut T { /* ... */ }
}

let boxed_str: Box<str> = /* ... */;
let ptr = Box::into_raw(boxed_str);
```

如果把 `into_raw` 定义为固有方法[^inherent-method] 
([inherent method][inherent methods]) ，
那么在调用它的时候，很难分清这个方法是来自于 `Box<T>` 的
还是 来自于 `T` 的。[^C-SMART-PTR]

```rust,ignored
impl<T> Box<T> where T: ?Sized {
    // 别这样定义
    fn into_raw(self) -> *mut T { /* ... */ }
}

let boxed_str: Box<str> = /* ... */;

// 这个方法来自于 `str` ，通过智能指针的 `Deref` 做到的
boxed_str.chars()

// 但这个方法是来自于 `Box<str>` 的...吗？（需要去翻阅文档验证）
boxed_str.into_raw()
```

[^inherent-method]: 译者注：Rust 里没有继承 (inherence) 的概念，
可参阅 [Rust Book: inherence] 一节。\
但是有 **inherent** 的概念，比如 [inherent methods] 、[inherent implementations] ，
一般对这个形容词翻译为 “[固有（的）][固有的]” 。\
然而 inherent methods = [associated (or staic) functions] + [methods] ，
所以我认为这句的 inherent method 应该是指 [method][methods] （带 `self` 参数的函数）。

[^C-SMART-PTR]: 译者注：这个原则是提醒大家，智能指针的固有方法一般以关联函数形式定义，
但也不是绝对没有方法，比如 [`RC::downcast`] 的参数就是 `self` 。


[`RC::downcast`]: http://129.28.186.100/rust-docs/rust/html/std/rc/struct.Rc.html#method.downcast
[methods]: https://doc.rust-lang.org/nightly/reference/items/associated-items.html#methods
[associated (or staic) functions]: https://doc.rust-lang.org/nightly/reference/items/associated-items.html#associated-functions-and-methods
[inherent methods]: http://doc.rust-lang.org/nightly/reference/glossary.html#inherent-method
[inherent implementations]: https://doc.rust-lang.org/nightly/reference/items/implementations.html#inherent-implementations
[固有的]: https://minstrel1977.gitee.io/rust-reference/items/implementations.html#inherent-implementations
[Rust Book: inherence]: https://doc.rust-lang.org/book/ch17-01-what-is-oo.html?highlight=inherence#inheritance-as-a-type-system-and-as-code-sharing

<a id="c-conv-specific"></a>
## 类型转换的重点应放在涉及类型中最明确的类型上 

> Conversions live on the most specific type involved (C-CONV-SPECIFIC)

在拿不准怎么转换类型的时候，相比于使用 `from_` ，建议优先使用
`to_` / `as_` / `into_` 。因为后者更人性化，
而且可与其他方法一起被链式调用。

对于转换涉及的两种类型，很多时候 其中一种类型更清晰明确：
这种类型 有某些额外的不变式 (invariant) 或 表现出其他类型不具备的特点。
比如 [`str`] 就比 `&[u8]` 更明确，因为前者是 UTF-8 编码的字节序列。

[`str`]: https://doc.rust-lang.org/std/primitive.str.html

转换应该重点放在所涉及类型中更明确的类型上。
所以 `str` 提供了 [`as_bytes`] 方法和 [`from_utf8`] 构造函数，
从而实现从 `str` 转换成 `&[u8]` 值、从 `&[u8]` 转换成 `str` 。
除了更直观，这种做法避免用无数转换方法 污染 `&[u8]` 这样的具体类型。


[`as_bytes`]: https://doc.rust-lang.org/std/primitive.str.html#method.as_bytes
[`from_utf8`]: https://doc.rust-lang.org/std/str/fn.from_utf8.html


<a id="c-method"></a>
## 有清楚接收者的函数应写成方法的形式 

> Functions with a clear receiver are methods (C-METHOD)

如果一个函数操作与具体对象之间具有清楚而密切的关系，那么建议这样做：

```rust,ignored
impl Foo {
    pub fn frob(&self, w: widget) { /* ... */ }
}
```

而不建议这样做：

```rust,ignored
pub fn frob(foo: &Foo, w: widget) { /* ... */ }
```

方法 ([methods]) 比函数有着巨大的优势：

* 方法可以直接被调用，无需导入或者验证资格：你需要的就只是一个合适的类型的值。
* 调用方法会自动借用（包括可变借用）[^auto-refer]。
* 方法可以轻松回答 “如何处理具有类型 `T` 的值” 的问题（在 rustdoc 中尤其有用）。
* 方法提供的 `self` 标注在传达所有权关系上更加简明和清晰。

[^auto-refer]: 译者注：如果你不明白这句话，可以阅读 
[Rust Book: Where’s the -> Operator?](https://doc.rust-lang.org/book/ch05-03-method-syntax.html#wheres-the---operator)

<a id="c-no-out"></a>
## 函数不该把返回值作为其参数 

> Functions do not take out-parameters (C-NO-OUT)

返回多个 `Bar` 类型的值，建议写成[^C-NO-OUT]：

```rust,ignored
fn foo() -> (Bar, Bar)
```

而不写成：

```rust,ignored
fn foo(output: &mut Bar) -> Bar
```

返回元组和结构体这样的复合类型是被编译器高效处理的，而且不需要堆分配。
如果函数需要返回多个值，那么应该借助这些复合类型。

一个重要的反例[^C-NO-OUT2]：
函数的目的是修改已有数据的值，那么不适用于这条原则。
比如重新使用缓冲数据：

```rust,ignored
fn read(&mut self, buf: &mut [u8]) -> io::Result<usize>
```

[^C-NO-OUT]: 译者注：这个例子大概是说，如果函数返回的值与外界没什么关系，
就无需传入函数。比如函数返回完全在其内部处理产生的 `&'static str` ，
那么就不需要给函数传入 `&mut str` 。

[^C-NO-OUT2]: 译者注：再举一个常见的反例，外部预先定义空字符串，然后传入函数，
函数通过可变引用已经处理了输入，再返回这个字符串的话就是不必要的了。
所以这个原则的目的可能是让读者处理函数输入与输出的时候，弄清楚真正的意图，
你需要所有权的数据还是修改数据，从而采用高效的做法。

<a id="c-overload"></a>
## 重载运算符不足为奇 

> Operator overloads are unsurprising (C-OVERLOAD)

内置的运算符语法（ `*` 、 `|` 等等）可以通过实现 [`std::ops`] 里的 trait
使其作用于任意类型。这些运算符带有明确的目的：
`Mul` trait 只应用于类似于乘法的操作
（而且具有某些性质，像可结合性这样的运算性质） ，
其他 trait 也是类似的。

[`std::ops`]: https://doc.rust-lang.org/std/ops/index.html#traits


<a id="c-deref"></a>
## 只对智能指针实现 `Deref` 和 `DerefMut` trait 

> Only smart pointers implement `Deref` and `DerefMut` (C-DEREF)

很多情况下，编译器会隐式使用 `Deref` trait 来与方法做交互。
与之有关的规则是专门为适应智能指针而设计的，
所以这个 trait 只能应用于智能指针。

来自标准库的例子：

- [`Box<T>`](https://doc.rust-lang.org/std/boxed/struct.Box.html)
- [`String`](https://doc.rust-lang.org/std/string/struct.String.html)
  是 [`str`](https://doc.rust-lang.org/std/primitive.str.html) 的智能指针
- [`Rc<T>`](https://doc.rust-lang.org/std/rc/struct.Rc.html)
- [`Arc<T>`](https://doc.rust-lang.org/std/sync/struct.Arc.html)
- [`Cow<'a, T>`](https://doc.rust-lang.org/std/borrow/enum.Cow.html)


<a id="c-ctor"></a>
## 构造函数是静态的、固有的方法 

> Constructors are static, inherent methods (C-CTOR)

在 Rust 中，构造函数 (constructor) 是一种惯例。
关于构造函数，也有各种习惯性做法，
这些做法之间有着细微的区别。

1. 最基础的构造函数是 没有任何参数的 `new` 命名的形式：

```rust,ignored
impl<T> Example<T> {
    pub fn new() -> Example<T> { /* ... */ }
}
```

构造函数是某个类型静态的（不带 `self` 参数）、固有的方法，
用来构造（或者说 实例化 ）该类型数据。
构造函数通常与完整导入类型名称一起工作，从而让构造过程清晰而简明。

```rust,ignored
use example::Example;

// Construct a new Example.
let ex = Example::new();
```

`new` 函数一般应该作为实例化类型的基础方法。
有时像上面一样不带有任何参数，有时也会像 [`Box::new`] 需要放入`Box` 的值那样
那样带有参数。

2. 有些类型的构造函数，尤其 I/O 资源类型里的绝大多数类型，
习惯上给构造函数取区分性很强的名字。\
比如 [`File::open`] 、 [`Mmap::open`] 、
[`TcpStream::connect`] 和 [`UdpSocket::bind`] 。
在专门领域内取这些名字是合适的。

3. 有时会有多种方式构造一个类型。这种情况下，像 [`Mmap::open_with_offset`] 那样
  使用 `_with_foo` 后缀作为二级构造函数是很常见的。\
但是如果你的类型需要各种初始化配置 (construction options) ，
那就考虑使用 构造模式 ([builder pattern][C-BUILDER]) 吧。

4. 有些构造函数是 “类型转换构造器” (conversion constructors) ，
即 从已有的类型的值生成新的另外一个类型的值。
这种构造函数的名字往往以 `from_` 开头，
就像 [`std::io::Error::from_raw_os_error`] 那样。
注意，虽然这与 [`From`][C-CONV-TRAITS] trait 十分相像，
但是它们之间有三个不同点：

- `from_` 构造函数可能是 unsafe 的；而 `From` trait 不可能是 unsafe 的。
  一个例子是 [`Box::from_raw`] 。
- `from_` 构造函数可以接收额外的参数来让源数据转换方式更加清楚，正如 
  [`u64::from_str_radix`] 。
- `From` trait 只适合源数据类型足够确定输出数据类型编码的情况。
  当输入一大堆位数据，使用 [`u64::from_be`] 或 [`String::from_utf8`]
  这样的类型转换函数才能知道数据的类型。

[`Box::from_raw`]: https://doc.rust-lang.org/std/boxed/struct.Box.html#method.from_raw
[`u64::from_str_radix`]: https://doc.rust-lang.org/std/primitive.u64.html#method.from_str_radix
[`u64::from_be`]: https://doc.rust-lang.org/std/primitive.u64.html#method.from_be
[`String::from_utf8`]: https://doc.rust-lang.org/std/string/struct.String.html#method.from_utf8

5. 注意，同时实现 `Default` trait 和 `new` 构造函数是常见而必要的。
如果都用在一个类型上，那么它们所做的事情应该一模一样。
可以根据其中一个来实现另一个。

[C-BUILDER]: type-safety.html#c-builder
[C-CONV-TRAITS]: interoperability.html#c-conv-traits

来自标准库的例子：

- [`std::io::Error::new`] 常用来构造 IO 错误
- [`std::io::Error::from_raw_os_error`] 
  是一个基于操作系统错误代码的转换构造器 
- [`Box::new`] 生成新的容器类型，需要一个参数 
- [`File::open`] 打开一个文件资源
- [`Mmap::open_with_offset`] 打开映射到内存的文件，支持附加的配置选项

[`File::open`]: https://doc.rust-lang.org/stable/std/fs/struct.File.html#method.open
[`Mmap::open`]: https://docs.rs/memmap/0.5.2/memmap/struct.Mmap.html#method.open
[`Mmap::open_with_offset`]: https://docs.rs/memmap/0.5.2/memmap/struct.Mmap.html#method.open_with_offset
[`TcpStream::connect`]: https://doc.rust-lang.org/stable/std/net/struct.TcpStream.html#method.connect
[`UdpSocket::bind`]: https://doc.rust-lang.org/stable/std/net/struct.UdpSocket.html#method.bind
[`std::io::Error::new`]: https://doc.rust-lang.org/std/io/struct.Error.html#method.new
[`std::io::Error::from_raw_os_error`]: https://doc.rust-lang.org/std/io/struct.Error.html#method.from_raw_os_error
[`Box::new`]: https://doc.rust-lang.org/stable/std/boxed/struct.Box.html#method.new

