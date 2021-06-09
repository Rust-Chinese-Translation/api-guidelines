# Rust API 编写指南

这是一组关于如何设计和呈现 Rust APIs 的建议。
这些建议主要由 Rust library 团队编写，
总结了 Rust 生态下构建标准库和其他 crates 的经验。

这些建议只是给你指导。
有些指导建议相比而言需要严格遵守。
有些不那么严格，因为那些是模糊的、仍然在发展完善中的。

Rust crate 的编写者应该站在符合 Rust 语言习惯、且相互协同的开发角度，
慎重考虑和采纳他们认为合适的建议。\
虽然 crate 编写者可能觉得 **采纳** 这些建议的 crates 比那些
**不采纳** 这些建议的 crates 会更好地融入现有的 crate 生态，
但是这些指导建议不应该被认为是 crate 作者所必须遵守的。

这本书分为两部分：
1. 简明的 [一览表][checklist] ：罗列了所有单独的原则[^guidelines]，
适合在检查修改 crate 时进行浏览。
2. 专题章节 (topical chapters) 详细解释了这些原则。

如果你对贡献 API 编写指南感兴趣，可以看看 [贡献说明][contributing.md]、
加入我们的 [Gitter 频道][Gitter channel] 。[^project]

[checklist]: checklist.html
[contributing.md]: https://github.com/rust-lang/api-guidelines/blob/master/CONTRIBUTING.md
[Gitter channel]: https://gitter.im/rust-impl-period/WG-libs-guidelines
[^guidelines]: 译者注：鉴于这本书的 *guidelines* 是 “**建议**” 而非 “必须” ，
所以翻译成 “原则” 语义偏重，但是译者暂时没想到此处有比 “原则” 更通顺而精简的词语。\
此外，为了语句通顺， *guidelines* 在不同的语境下会翻译成 “指南” 、 “编写指南” 、
“指导” 、 “指导建议” 、 “原则” 之类的同义词。

[^project]: 译者注：
[原项目地址](https://github.com/rust-lang/api-guidelines) |
[中文渲染版](https://zjp-cn.github.io/api-guidelines) |
[国内站点](http://129.28.186.100/api-guidelines)

