# 灵活性


<a id="c-intermediate"></a>
## 为避免重复计算 函数应提供中间结果 

> Functions expose intermediate results to avoid duplicate work (C-INTERMEDIATE)

很多函数为了解决问题，会计算一些有趣且有关的数据。
如果这些数据可能引发使用者的兴趣，
请考虑在 API 里面返回它们。

来自标准库的例子：

- [`Vec::binary_search`] 
  无论值有没有找到，它都不返回 `bool` 值，也不返回 `Option<usize>` 
  来表明可能找到的索引位置。实际上，在找到值的时候，它返回值的索引；
  在没找到值的时候，它返回需要插入这个值的位置。

- [`String::from_utf8`] 
  若传入的字节不是 UTF-8 的话，它运行失败，然后返回中间结果：
  提供输入字节中第一个无效 UTF-8 序列的索引，也可以返回输入字节的所有权。

- [`HashMap::insert`] 
  返回 `Option<T>` ，如果预先存在一个值，那么返回这个值。
  使用者如果想恢复插入操作之前的值，那么返回的值就避免用户二次查找哈希表了。

[`Vec::binary_search`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.binary_search
[`String::from_utf8`]: https://doc.rust-lang.org/std/string/struct.String.html#method.from_utf8
[`HashMap::insert`]: https://doc.rust-lang.org/stable/std/collections/struct.HashMap.html#method.insert


<a id="c-caller-control"></a>
## 调用方决定在何处复制和替换数据  

> Caller decides where to copy and place data (C-CALLER-CONTROL)

如果函数参数需要具有所有权，
那么直接获取所有权，而不要通过借用和复制的方式来获取所有权。[^C-CALLER-CONTROL]

```rust,ignored
// 应该该这样做
fn foo(b: Bar) {
    /* 直接使用 `b` 的所有权 */
}

// 不要这样做
fn foo(b: &Bar) {
    let b = b.clone();
    /* 复制之后再拿到 `b` 的所有权 */
}
```

如果函数参数不需要所有权，那就获取共享引用或独占引用，
不要获取所有权，然后把数据扔掉。

```rust,ignored
// 应该该这样做
fn foo(b: &Bar) {
    /* 使用借用 */
}

// 不要这样做
fn foo(b: Bar) {
    /* 使用借用，但是函数进行返回的时候，偷偷把 `b` 给 drop 掉了 */
}
```
`Copy` trait 应该在真正需要它的时候才使用它，
不要把它当做低成本复制的方式。

[^C-CALLER-CONTROL]: 译者注：虽然其内容讲的是被调用方（函数），
但是这条原则是站在使用者 (caller) 角度来描述的。
因为用户可以选择（也可以不选择）复制一份具有所有权的数据再传入需要所有权的函数，
这个复制数据的选择权（决定权）在于调用方。
函数不应该做这个决定。

<a id="c-generic"></a>
## 函数通过泛型来对参数做最小范围的假设 

> Functions minimize assumptions about parameters by using generics (C-GENERIC)

对函数输入做越小范围的假设，
函数的使用场景就越广泛：

如果函数只需要迭代类型的数据，请这样写：

```rust,ignored
fn foo<I: IntoIterator<Item = i64>>(iter: I) { /* ... */ }
```

而不要详细到这般：

```rust,ignored
fn foo(c: &[i64]) { /* ... */ }
fn foo(c: &Vec<i64>) { /* ... */ }
fn foo(c: &SomeOtherCollection<i64>) { /* ... */ }
```

一般来说，考虑使用泛型来准确表明函数对参数的假设关系是什么。

### 泛型的优点

* **可复用**：泛型函数能应用在广泛的类型上，同时明确给出了这些类型的必须满足的关系。
* **静态分派和编译器优化**：
  每个泛型函数都被专门用于实现了 trait bounds 的具体的类型 
  （即 单态化 [monomorphized] ），这意味着：
   1. 调用的 trait 方法是静态生成的，因此是直接对 trait 实现的调用
   2. 编译器能对这些调用做内联 (inline) 和其他优化
* **内联式布局**：如果结构体和枚举体类型具有某个泛型参数 `T` ，
  `T` 的值将在结构体和枚举体里以内联方式排列，不产生任何间接调用。
* **可推断**：由于泛型函数的类型参数通常是推断出来的，
  泛型函数可以减少复杂的代码，比如显式转换、通常必须的一些方法调用。
* **精确的类型**：因为泛型给实现了某个 trait 的具体类型一个名称，
  从而有可能清楚这个类型需要或创建的地方在哪。比如这个函数：

  ```rust,ignored
  fn binary<T: Trait>(x: T, y: T) -> T
  ```

  会保证消耗和创建具有相同类型 `T` 的值；不可能传入实现了 `Trait` 
  的但不同名称的两个类型。

[monomorphized]: https://doc.rust-lang.org/book/ch10-01-syntax.html#performance-of-code-using-generics

### 泛型的缺点

* **增加代码大小**：单态化泛型函数意味着函数体会被复制。
  增加代码大小和静态分派的性能优势之间必须做出衡量。
* **类型同质化**：这是 “精确的类型” 带来的另一面：
  如果 `T` 是类型参数，那么它代表一个单独的实际类型。
  对于像 `Vec<T>` 这样具体的单独的元素类型也是一样，
  而且 `Vec` 实际上为了内联这些元素，进行了专门的处理。
  有时候，不同的类型会更有用，参考 [trait objects][C-OBJECT] 。
* **签名冗余**：过度使用泛型会造成阅读和理解函数签名更困难。

[C-OBJECT]: #c-object

来自标准库的例子：

- [`std::fs::File::open`] 以泛型 `AsRef<Path>` 作为参数。
  它能方便根据 `"f.txt"` 这样的字符串字面值、 [`Path`] 、 [`OsString`] 
  以及其他一些类型中打开文件。

[`std::fs::File::open`]: https://doc.rust-lang.org/std/fs/struct.File.html#method.open
[`Path`]: https://doc.rust-lang.org/std/path/struct.Path.html
[`OsString`]: https://doc.rust-lang.org/std/ffi/struct.OsString.html


<a id="c-object"></a>
## trait 用作 object 时应当是安全的 

> Traits are object-safe if they may be useful as a trait object (C-OBJECT)

trait object 有一些很重要的限制：
1. 通过 trait object 调用的方法不能使用泛型；
2. 除了接收者位置上可以使用 `Self`，其他地方不能使用 `Self` （比如 返回值类型）。

设计 trait 的时候，早些决定这个 trait 要作为 object 还是作为泛型的 bound 来使用。

如果 trait 用作 object ，它的方法应该被传给和返回 trait objects ，
而不是泛型。

带有 `Self: Sized` 的 `where` 语句可以用来把某个具体的方法从 trait object 
里排除掉 (exclude) 。下面这个 trait 不是安全的 ([object-safe]) ，
因为具有泛型方法。

```rust,ignored
trait MyTrait {
    fn object_safe(&self, i: i32);

    fn not_object_safe<T>(&self, t: T);
}

fn f() -> Box<dyn MyTrait> { /* 代码 */ }
```

增加所需的 `Self: Sized` 来把这个泛型方法从 trait object 里排除掉，
从而让 trait 是安全的。

```rust,ignored
trait MyTrait {
    fn object_safe(&self, i: i32);

    fn not_object_safe<T>(&self, t: T) where Self: Sized;
}

fn f() -> Box<dyn MyTrait> { /* 代码 */ }
```

### trait objects 的优点

* **异质性**：当你需要 trait object 的时候，的确真的需要它。
* **代码体积小**：不像泛型， trait objects 不生成处理过的代码（单态化），
  所以能很大程度减少代码体积。

### trait objects 的缺点

* **无泛型方法**： trait objects 现在无法提供泛型方法。
* **动态分派和胖指针**： trait objects 天生就涉及间接操作，所以具有性能惩罚。
* **没有 `Self`** ： 除了接收者位置上可以使用，其他地方不能使用 `Self` 类型。

来自标准库的例子：

- [`io::Read`] 和 [`io::Write`] trait 常用作 trait objects
- [`Iterator`] trait 有多个具有 `where Self: Sized` 标记的泛型方法，
  目的是能让 `Iterator` 用作 trait object

[`io::Read`]: https://doc.rust-lang.org/std/io/trait.Read.html
[`io::Write`]: https://doc.rust-lang.org/std/io/trait.Write.html
[`Iterator`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html

[object-safe]: https://doc.rust-lang.org/nightly/reference/items/traits.html#object-safety
