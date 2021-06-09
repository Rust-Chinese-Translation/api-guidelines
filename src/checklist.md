# Rust API 指导清单

<!-- Read CONTRIBUTING.md before writing new guidelines -->

- **命名** ( *crate 遵照 Rust 命名规范* )
  - 大小写规范 RFC 430 ([C-CASE])
  - 遵循 `as_`, `to_`, `into_` 规范 用以特定类型转换 ([C-CONV])
  - getter 命名规范 ([C-GETTER])
  - 遵循 `iter`, `iter_mut`, `into_iter` 规范 用以生成迭代器 ([C-ITER])
  - 生成迭代器的方法与迭代器类型同名 ([C-ITER-TY])
  - cargo feature 名中不应该有无意义的词 ([C-FEATURE])
  - 词性顺序一致 ([C-WORD-ORDER])
- **互通互用** ( *crate 很好地与其他库提供的功能进行交互* )
  - 类型应尽早实现常见的 traits ([C-COMMON-TRAITS])
    - `Copy`, `Clone`, `Eq`, `PartialEq`, `Ord`, `PartialOrd`, `Hash`, `Debug`,
      `Display`, `Default`
  - 使用 `From`, `AsRef`, `AsMut` trait 来转换类型 ([C-CONV-TRAITS])
  - 给集合实现 `FromIterator` 和 `Extend` trait ([C-COLLECT])
  - 给数据结构实现 Serde 的 `Serialize` 和 `Deserialize` trait ([C-SERDE])
  - 类型应尽可能实现 `Send` 和 `Sync` trait ([C-SEND-SYNC])
  - Error 类型 十分直观和有用 ([C-GOOD-ERR])
  - 二进制数类型应提供 `Hex`, `Octal`, `Binary` 的格式化方式 ([C-NUM-FMT])
  - reader/writer 泛型函数使用 `R: Read` 和 `W: Write` 参数传值 ([C-RW-VALUE])
- **宏** ( *crate 应展现良好的宏* )
  - 输入语法与输出语法一致 ([C-EVOCATIVE])
  - 宏与属性形成有机的整体 ([C-MACRO-ATTR])
  - 生成条目的宏可以在条目被允许的地方使用 ([C-ANYWHERE])
  - 生成条目的宏应支持可视分类符 ([C-MACRO-VIS])
  - 类型分类符 `$t:ty` 是灵活的 ([C-MACRO-TY])
- **文档编写** ( *crate 有丰富的文档说明* )
  - crate 级别的文档应该详实有例 ([C-CRATE-DOC])
  - 每个条目都应该有例子 ([C-EXAMPLE])
  - 例子应该使用 `?` 而不使用 `try!` 或者 `unwrap` ([C-QUESTION-MARK])
  - 函数涉及错误、 panic、安全性时 应该加以说明 ([C-FAILURE])
  - 给相关的内容添加超链接 ([C-LINK])
  - Cargo.toml 应包含所有常见的配置数据 ([C-METADATA])
    - 作者、描述、版权、主页、文档、仓库、readme、关键词、分类
  - 设置 html_root_url 属性 "https://docs.rs/CRATE/X.Y.Z" ([C-HTML-ROOT])
  - 发布时 记录该版本的重大变化 ([C-RELNOTES])
  - 文档不应该展示无太大帮助的实现细节 ([C-HIDDEN])
- **可预测** ( *crate 让清晰可读的代码正如它展示的那样工作* )
  - 智能指针不增加固有方法 ([C-SMART-PTR])
  - 类型转换的重点应放在涉及类型中最明确的类型上 ([C-CONV-SPECIFIC])
  - 有清楚接收者的函数应写成方法的形式 ([C-METHOD])
  - 函数不该把返回值作为其参数 ([C-NO-OUT])
  - 重载运算符不足为奇 ([C-OVERLOAD])
  - 只对智能指针实现 `Deref` 和 `DerefMut` trait ([C-DEREF])
  - 构造函数是静态的、固有的方法 ([C-CTOR])
- **灵活性** ( *crate 应支持现实中各种各样的使用场景* )
  - 为避免重复计算 函数应提供中间结果 ([C-INTERMEDIATE])
  - 调用方决定在何处复制和替换数据 ([C-CALLER-CONTROL])
  - 函数通过泛型来对参数做最小范围的假设 ([C-GENERIC])
  - trait 用作 object 时应当是安全的 ([C-OBJECT])
- **类型安全** ( *crate 应有效地利用类型系统* )
  - newtype 提供静态的区分功能 ([C-NEWTYPE])
  - 参数应使用类型来表明意图 ([C-CUSTOM-TYPE])
  - 用 `bitflags` 来存放一组标志 ([C-BITFLAG])
  - 利用构造模式构造复杂的值 ([C-BUILDER])
- **可依赖** ( *crate 不太可能出错* )
  - 函数会验证其参数 ([C-VALIDATE])
  - 析构函数不该运行失败 ([C-DTOR-FAIL])
  - 阻塞的析构函数应有替代的办法 ([C-DTOR-BLOCK])
- **可调试** ( *crate 易于调试* )
  - 所有公有的类型都应该实现 `Debug` ([C-DEBUG])
  - `Debug` 呈现的内容永远不为空 ([C-DEBUG-NONEMPTY])
- **前瞻性** ( *crate 能在不破坏使用者代码的情况下随时改进* )
  - 封装的 traits 隔绝下游的实现 ([C-SEALED])
  - 结构体具有私有字段 ([C-STRUCT-PRIVATE])
  - newtypes 封装起实现的细节 ([C-NEWTYPE-HIDE])
  - 已经 `derive` 的数据结构不应该再使用 trait bounds ([C-STRUCT-BOUNDS])
- **必要项** ( *对于使用者来说，这真的很重要* )
  - 稳定版 crate 必须具有稳定的公有依赖 ([C-STABLE])
  - crate 及其依赖必须有许可证 ([C-PERMISSIVE])


[C-CASE]: naming.html#c-case
[C-CONV]: naming.html#c-conv
[C-GETTER]: naming.html#c-getter
[C-ITER]: naming.html#c-iter
[C-ITER-TY]: naming.html#c-iter-ty
[C-FEATURE]: naming.html#c-feature
[C-WORD-ORDER]: naming.html#c-word-order

[C-COMMON-TRAITS]: interoperability.html#c-common-traits
[C-CONV-TRAITS]: interoperability.html#c-conv-traits
[C-COLLECT]: interoperability.html#c-collect
[C-SERDE]: interoperability.html#c-serde
[C-SEND-SYNC]: interoperability.html#c-send-sync
[C-GOOD-ERR]: interoperability.html#c-good-err
[C-NUM-FMT]: interoperability.html#c-num-fmt
[C-RW-VALUE]: interoperability.html#c-rw-value

[C-EVOCATIVE]: macros.html#c-evocative
[C-MACRO-ATTR]: macros.html#c-macro-attr
[C-ANYWHERE]: macros.html#c-anywhere
[C-MACRO-VIS]: macros.html#c-macro-vis
[C-MACRO-TY]: macros.html#c-macro-ty

[C-CRATE-DOC]: documentation.html#c-crate-doc
[C-EXAMPLE]: documentation.html#c-example
[C-QUESTION-MARK]: documentation.html#c-question-mark
[C-FAILURE]: documentation.html#c-failure
[C-LINK]: documentation.html#c-link
[C-METADATA]: documentation.html#c-metadata
[C-HTML-ROOT]: documentation.html#c-html-root
[C-RELNOTES]: documentation.html#c-relnotes
[C-HIDDEN]: documentation.html#c-hidden

[C-SMART-PTR]: predictability.html#c-smart-ptr
[C-CONV-SPECIFIC]: predictability.html#c-conv-specific
[C-METHOD]: predictability.html#c-method
[C-NO-OUT]: predictability.html#c-no-out
[C-OVERLOAD]: predictability.html#c-overload
[C-DEREF]: predictability.html#c-deref
[C-CTOR]: predictability.html#c-ctor

[C-INTERMEDIATE]: flexibility.html#c-intermediate
[C-CALLER-CONTROL]: flexibility.html#c-caller-control
[C-GENERIC]: flexibility.html#c-generic
[C-OBJECT]: flexibility.html#c-object

[C-NEWTYPE]: type-safety.html#c-newtype
[C-CUSTOM-TYPE]: type-safety.html#c-custom-type
[C-BITFLAG]: type-safety.html#c-bitflag
[C-BUILDER]: type-safety.html#c-builder

[C-VALIDATE]: dependability.html#c-validate
[C-DTOR-FAIL]: dependability.html#c-dtor-fail
[C-DTOR-BLOCK]: dependability.html#c-dtor-block

[C-DEBUG]: debuggability.html#c-debug
[C-DEBUG-NONEMPTY]: debuggability.html#c-debug-nonempty

[C-SEALED]: future-proofing.html#c-sealed
[C-STRUCT-PRIVATE]: future-proofing.html#c-struct-private
[C-NEWTYPE-HIDE]: future-proofing.html#c-newtype-hide
[C-STRUCT-BOUNDS]: future-proofing.html#c-struct-bounds

[C-STABLE]: necessities.html#c-stable
[C-PERMISSIVE]: necessities.html#c-permissive
