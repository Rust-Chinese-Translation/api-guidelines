# 类型安全


<a id="c-newtype"></a>
## newtype 提供静态的区分功能 

> Newtypes provide static distinctions (C-NEWTYPE)

newtype 能静态地[^static]区分同一个里层类型不同的含义。

比如 一个 `f64` 的值可能用来表示几英里，也可能表示几公里。
使用 newtype ，就能让我们知道这个值意图表示什么。

```rust,ignored
struct Miles(pub f64);
struct Kilometers(pub f64);

impl Miles {
    fn to_kilometers(self) -> Kilometers { /* ... */ }
}
impl Kilometers {
    fn to_miles(self) -> Miles { /* ... */ }
}
```

一旦对这两种类型作区分，我们能静态地确保不把他们混为一谈。
打个比方，下面这个函数不节能意外地调用 `Kilometers` 类型的值。

```rust,ignored
fn are_we_there_yet(distance_travelled: Miles) -> bool { /* ... */ }
```

编译器会提醒我们需要转换成 `Miles` ，
从而避免了某些 [灾难性的 bugs][catastrophic bugs] 。

[catastrophic bugs]: http://en.wikipedia.org/wiki/Mars_Climate_Orbiter

[^static]: 译者注：之所以称之为 *static* / *statically* ，是因为 newtype 
类型一旦被定义就不会发生变化，从而它代表的含义就是固定不变的。

<a id="c-custom-type"></a>
## 参数应使用类型来表明意图 

> Arguments convey meaning through types, not `bool` or `Option` (C-CUSTOM-TYPE)

请写这样的代码：

```rust,ignored
let w = Widget::new(Small, Round)
```

而不是这样的：

```rust,ignored
let w = Widget::new(true, false)
```

像 `bool` 、 `u8` 、 `Option` 这样的核心类型有很多可能的含义。

所以无论你使用枚举体、结构体还是元组，
请经过思考定义一种类型来传达含义和 [不变性][invariants]。
上面这个例子中，如果不看参数的名字，根据 `true` 和 `false` 
你不能马上知道它们是什么意思，但是 `Small` 和 `Round` 就更具有联想性。

使用自定义类型让后面要谈到的配置拓展更简单，比如增加一个 `ExtraLarge` 成员。

阅读 [newtype 模式][C-NEWTYPE] 来做到以低花销的方式封装现有类型。

[invariants]: https://doc.rust-lang.org/nightly/reference/subtyping.html#variance

[C-NEWTYPE]: #c-newtype


<a id="c-bitflag"></a>
## 用 `bitflags` 来存放一组标志 

> Types for a set of flags are `bitflags`, not enums (C-BITFLAG)

Rust 支持 `enum` 类型，它可以带有显式的判别式 (discriminants) 。

```rust,ignored
enum Color {
    Red = 0xff0000,
    Green = 0x00ff00,
    Blue = 0x0000ff,
}
```

当枚举类型需要被序列化成整数以便兼容其他系统或语言的时候，自定义判别式是有用的。
判别式提供了 “类型安全” 的 APIs ：
给函数传入 `Color` 而不是整数，就能保证得到良好形式的输入，
即使函数把这些 `Color` 输入仍然看作是整数。

`enum` 允许 API 从其中刚好选择一个成员。
有时 API 输入存在或需要一组标志 (flags) ，在 C 语言里，
经常让每个标志 (flag) 对应上一个特定的位 (bit) ，
比如让一个数字代表是 32 位或 64 位。
Rust 的 [`bitflags`] crate 为这种模式提供了类型安全的解决办法。

[`bitflags`]: https://github.com/bitflags/bitflags

```rust,ignored
#[macro_use]
extern crate bitflags;

bitflags! {
    struct Flags: u32 {
        const FLAG_A = 0b00000001;
        const FLAG_B = 0b00000010;
        const FLAG_C = 0b00000100;
    }
}

fn f(settings: Flags) {
    if settings.contains(Flags::FLAG_A) {
        println!("doing thing A");
    }
    if settings.contains(Flags::FLAG_B) {
        println!("doing thing B");
    }
    if settings.contains(Flags::FLAG_C) {
        println!("doing thing C");
    }
}

fn main() {
    f(Flags::FLAG_A | Flags::FLAG_C);
}
```


<a id="c-builder"></a>
## 利用构造模式构造复杂的值 

> Builders enable construction of complex values (C-BUILDER)

有些数据结构构造起来很复杂，因为构造它们需要：

* 大量的输入
* 复合类型（比如 slices ）
* 可选择的配置数据
* 在多个可选项中做选择

这很容易导致许多带有不同的构造器 (constructors) ，而且每个构造器有很多参数。

假设 `T` 是上面描述的一种数据结构，那么考虑给 `T` 引入构造器模式 (builder pattern) 。

1. 引入单独的数据类型 `TBuilder` 来增量配置 `T` 的值。
   如果可以的话，选一个更好听的名字：比如 [`Command`] 是
   子进程 [Child][child process] 的构造器、
   [`ParseOptions`] 是 [`Url`] 的构造器。
2. 仅当 `T` 需要数据的时候才应该给构造器的方法传入参数。
3. 构造器应该提供一套方便的配置方法，包括增量初始化复合型输入（像 slice 一样）。
   这些方法应该返回 `self` 从而允许链式调用。
4. 构造器应该提供一个或更多的最终方法来实际构造 `T` 。

[`Command`]: https://doc.rust-lang.org/std/process/struct.Command.html
[child process]: https://doc.rust-lang.org/std/process/struct.Child.html
[`Url`]: https://docs.rs/url/1.4.0/url/struct.Url.html
[`ParseOptions`]: https://docs.rs/url/1.4.0/url/struct.ParseOptions.html

构造器模式尤其适合于 构造 `T` 时涉及意外的连带后果 的场景，
比如开始一个任务或开启一个进程。

Rust 里有两种构造器模式，区别在于处理所有权上：

### 非消耗型构造器


有些情况构造最终的 `T` 不需要构造器被消耗掉 (consume) 。
下面这个例子是 [`std::process::Command`] 的简化版：

[`std::process::Command`]: https://doc.rust-lang.org/std/process/struct.Command.html

```rust,ignored
// 注意：这是一个简化的版本，实际中 Command API 不使用有所有权的 String

pub struct Command {
    program: String,
    args: Vec<String>,
    cwd: Option<String>,
    // etc
}

impl Command {
    pub fn new(program: String) -> Command {
        Command {
            program: program,
            args: Vec::new(),
            cwd: None,
        }
    }

    /// 增加一个命令行参数以传给程序
    pub fn arg(&mut self, arg: String) -> &mut Command {
        self.args.push(arg);
        self
    }

    /// 增加多个命令行参数以传给程序
    pub fn args(&mut self, args: &[String]) -> &mut Command {
        self.args.extend_from_slice(args);
        self
    }

    /// 设置子进程工作目录
    pub fn current_dir(&mut self, dir: String) -> &mut Command {
        self.cwd = Some(dir);
        self
    }

    /// 以子进程方式执行命令，然后返回子进程
    pub fn spawn(&self) -> io::Result<Child> {
        /* ... */
    }
}
```
注意 `spawn` 方法实际上利用构造器的配置来开启一个进程，
以共享引用的方式使用了构造器，
因为开启进程无需配置数据的所有权。
而且用于配置的方法传入和返回 `self` 的可变引用。

这种方式的优点：
整个过程都使用借用，`Command` 能方便地以一行或更复杂的方式使用。
**推荐** 使用这种方式。

```rust,ignored
// 用一行进行配置
Command::new("/bin/cat").arg("file.txt").spawn();

// 进行复杂的配置
let mut cmd = Command::new("/bin/ls");
cmd.arg(".");
if size_sorted {
    cmd.arg("-S");
}
cmd.spawn();
```

### 消耗型构造器

有时候，为了构造出最后的 `T` ，构造器必须传递所有权。
这意味着必须给方法传入 `self` 而不是 `&self` 。

```rust,ignored
impl TaskBuilder {
    /// 给将要运行的任务命名
    pub fn named(mut self, name: String) -> TaskBuilder {
        self.name = Some(name);
        self
    }

    /// 把本地任务的 stdout 重新定向
    pub fn stdout(mut self, stdout: Box<io::Write + Send>) -> TaskBuilder {
        self.stdout = Some(stdout);
        self
    }

    /// 创建和执行新的子任务
    pub fn spawn<F>(self, f: F) where F: FnOnce() + Send {
        /* ... */
    }
}
```

在这里， `stdout` 配置涉及到传传递 `io::Write` 的所有权，
所以必须在构造的时候（ `spawn` 方法里面）再传递给任务。

当最终的构造器需要所有权时，面临一个基本的抉择：

* 如果其他的构造方法被传入或者返回 可变借用 `&mut self` ，
  那么复杂的配置情况也能很好地工作，但是不可能用一行完成所有配置。
* 如果其他的构造方法被传入或者返回 有所有权的 `self` ，
  可以很好地使用一行来进行所有配置，但是在进行复杂配置时就不太方便。

遵循 “让简单的事简单完成，困难的事也可能完成” 这句箴言，
消耗型构造器所有的方法都应该被传入和返回有所有权的 `self` 。
用户的代码可以像下面这样工作：

```rust,ignored
// 用一行进行配置
TaskBuilder::new("my_task").spawn(|| { /* ... */ });

// 进行复杂的配置
let mut task = TaskBuilder::new();
task = task.named("my_task_2"); // 必须重新赋值出去，从而获得所有权
if reroute {
    task = task.stdout(mywriter);
}
task.spawn(|| { /* ... */ });
```

一行配置的方式像上一个方法那样可以工作，
因为所有权在每个构造器方法上依次传递，
直到被 `spawn` 消耗掉。
然而复杂配置的情况更啰嗦：
在最后一步需要把构造器重新赋值给一个变量。
