# GN 常见问题解答

<!-- [TOC] --> 
<!-- START doctoc --> 
<!-- END doctoc -->

## GN 文档在哪里?

GN 有广泛的内置帮助,所以你可以运行`gn help`,但你也可以看到所有的帮助[参考页面](reference.md).另见[快速开始](quick_start.md)指南和[语言和操作细节](language.md).

## 我可以生成 XCode 或 Visual Studio 项目吗?

您可以为 Xcode,Visual Studio,QTCreator 和 Eclipse 生成骨架(或包装器)项目,这些项目将列出构建中的文件和目标,但使用 Ninja 进行实际构建.你无法生成看起来像 GYP 本身的"真实"项目.

跑`gn help gen`更多细节.

## 如何生成常见的构建变体?

在 GN 中,args 使用构建目录而不是环境中的全局目录.编辑你的 args`out/Default`构建目录:

```
gn args out/Default
```

您可以在该文件中设置变量:

- 默认是调试版本.要做一个发布版本添加`is_debug = false`
- 默认值是静态构建.要进行组件构建添加`is_component_build = true`
- 默认是开发人员构建.要进行官方构建,请设置`is_official_build = true`
- 默认为 Chromium 品牌.要进行 Chrome 品牌塑造,请设置`is_chrome_branded = true`

## 我该如何进行交叉编译?

GN 为在单个构建中进行交叉编译和构建多个体系结构的东西提供了强大的支持.

看到[GNCrossCompiles](cross_compiles.md)了解更多信息.

## 我可以控制默认构建的目标吗?

是!如果在顶级(根)构建文件中创建名为"default"的组目标,即"//:default",GN 将告诉 Ninja 默认构建该目标,而不是构建所有内容.
