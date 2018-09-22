# GN 常见问题解答

<!-- [TOC] --> 
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [GN 文档在哪里?](#gn-%E6%96%87%E6%A1%A3%E5%9C%A8%E5%93%AA%E9%87%8C)
- [我可以生成 XCode 或 Visual Studio 项目吗?](#%E6%88%91%E5%8F%AF%E4%BB%A5%E7%94%9F%E6%88%90-xcode-%E6%88%96-visual-studio-%E9%A1%B9%E7%9B%AE%E5%90%97)
- [如何生成常见的构建变体?](#%E5%A6%82%E4%BD%95%E7%94%9F%E6%88%90%E5%B8%B8%E8%A7%81%E7%9A%84%E6%9E%84%E5%BB%BA%E5%8F%98%E4%BD%93)
- [我该如何进行跨平台编译?](#%E6%88%91%E8%AF%A5%E5%A6%82%E4%BD%95%E8%BF%9B%E8%A1%8C%E8%B7%A8%E5%B9%B3%E5%8F%B0%E7%BC%96%E8%AF%91)
- [我可以控制默认构建的目标吗?](#%E6%88%91%E5%8F%AF%E4%BB%A5%E6%8E%A7%E5%88%B6%E9%BB%98%E8%AE%A4%E6%9E%84%E5%BB%BA%E7%9A%84%E7%9B%AE%E6%A0%87%E5%90%97)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## GN 文档在哪里?

GN 有广泛的内置帮助,所以你可以运行`gn help`,但你也可以看[参考页面](reference.zh.md)得到所有的帮助.另见[快速开始](quick_start.zh.md)指南和[语言和操作细节](language.zh.md).

## 我可以生成 XCode 或 Visual Studio 项目吗?

您可以为 Xcode,Visual Studio,QTCreator 和 Eclipse 生成骨架(或包装器)项目,这些项目将列出构建中的文件和目标,但使用 Ninja 进行实际构建.你无法生成看起来像 GYP 本身的"真实"项目.

运行`gn help gen`更多细节.

## 如何生成常见的构建变体?

在 GN 中,`args` 使用构建目录而不是环境中的全局目录.编辑你的参数，在`out/Default`中构建目录:

```
gn args out/Default
```

您可以在该文件中设置变量:

- 默认是调试版本.要做一个发布版本添加`is_debug = false`
- 默认值是静态库构建.要进行组件构建添加`is_component_build = true`
- 默认是开发人员构建.要进行官方构建,请设置`is_official_build = true`
- 默认为 Chromium 品牌.要进行 Chrome 品牌塑造,请设置`is_chrome_branded = true`

## 我该如何进行跨平台编译?

GN 为在单个构建中进行跨平台编译和构建多个体系结构的东西提供了强大的支持.

看到[GNCrossCompiles](cross_compiles.zh.md)了解更多信息.

## 我可以控制默认构建的目标吗?

是! 如果在顶级(根)构建文件中创建名为"default"的目标组,即"//:default",GN 将告诉 Ninja 默认构建该目标, 而不是构建所有内容.
