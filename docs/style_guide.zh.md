# GN 风格指南

<!-- [TOC] --> 
<!-- START doctoc --> 
<!-- END doctoc -->

## 文件的命名和排序

### 构建文件的位置

与顶层更少的构建文件相比,让更多的构建文件更接近代码通常更意义的;这与我们使用 GYP 所做的是相反的.这使得事情更容易找到,也使得审计所需的所有者集合更小,因为更改更集中在特定的子目录.

### 目标

- 大多数构建文件应该有一个与目录同名的目标.这个目标应该是第一个目标.
- 其他目标应该按照某种逻辑顺序——通常更重要的目标将是第一,单元测试将遵循相应的目标.如果没有明确的排序,请考虑字母顺序.
- 测试支持库应该是名为"test\_support"的静态库.例如,"//ui/compositor:test\_support". 测试支持库应该包括非测试支持版本的公共库,因此只测试需要"test\_support"的目标(而不是两者).

命名建议

- 目标和配置应该使用小写字母和下划线分隔单词来命名,除非有充分的理由这样做.
- Source集合、groups和静态库不需要全局唯一名称.更喜欢给这些目标短而非冗余的名字,而不必担心全球的独功能.例如,写一个依赖项像`"//mojo/public/bindings"`要好得多,`而不是`"//mojo/public/bindings:mojo_bindings"`
- 共享库(以及扩展名、组件)必须具有全局唯一的输出名称.给这些目标非短且唯一的名称,然后提供全局唯一性的`output_name`给那个目标.
- Executables 和 tests 应该为全局唯一名称. 从技术上讲,只有输出名称必须是唯一的,而那因为输出名称只出现在 shell 和 bot 上,所以如果名称与执行文件出现的其他位置相一致,就不会那么混淆了.

### Configs

- 与单个目标相关联的配置应该被命名为与目标相同的配置,用`_config`.
- 配置应该在使用它的相应目标之前,立即出现.

### 例子

实例`src/foo/BUILD.gn`文件:

```bash
# Copyright 2016 The Chromium Authors. All rights reserved.
# Use of this source code is governed by a BSD-style license that can be
# found in the LICENSE file.

# Config for foo is named foo_config and immediately precedes it in the file.
config("foo_config") {
}

# Target matching path name is the first target.
executable("foo") {
}

# Test for foo follows it.
test("foo_unittests") {
}

config("bar_config") {
}

source_set("bar") {
}
```

## 目标内排序

1.  `output_name` / `visibility` / `testonly`
2.  `sources`
3.  `cflags`,`include_dirs`,`defines`,`configs`无论是什么顺序对你来说都是有意义的.
4.  `public_deps`
5.  `deps`

### 条件

仅影响一个变量(例如,添加单个 source 或为特定 OS 添加 flag)的简单条件可能低于它们影响的变量. 更复杂的条件会影响到一件事.

应以最小化,编写条件区块.

## 格式化和缩进

GN 包含一个内置的代码格式化程序,它定义了格式化风格.一些附加说明:

- 变量是`lower_case_with_underscores`.
- 注释应该是完整的句子,句号结尾.
- 编译器标志参数等,应该总是用它们所做的以及为什么需要来注释.

### Sources

更推荐只列出一次源. 有条件地包含源,而不是将它们全部列在顶部,然后在它们条件不适用时,排除它们.包含的条件通常会更清晰,因为文件只列出一次,并且在阅读时更容易推理.

```bash
  sources = [
    "main.cc",
  ]
  if (use_aura) {
    sources += [ "thing_aura.cc" ]
  }
  if (use_gtk) {
    sources += [ "thing_gtk.cc" ]
  }
```

### Deps

- Deps 应按字母顺序排列.
- 当前文件中的 Deps 应该首先写入,不符合的文件名不要.(仅`:foo`)
- 其他 deps 应该总是使用完全规范的路径名,除非由于某种原因需要相关的路径名.

```bash
  deps = [
    ":a_thing",
    ":mystatic",
    "//foo/bar:other_thing",
    "//foo/baz:that_thing",
  ]
```

### Import

使用完全合格的导入路径:

```bash
import("//foo/bar/baz.gni")  # Even if this file is in the foo/bar directory
```

## 用法

### 源集合与静态库

在大多数情况下,源集合和静态库可以互换使用.如果你不确定使用什么,源集合几乎从不出错,而且不太可能引起问题.

静态库遵循不同的链接规则.当链接中包含静态库时,只有包含未解析符号的对象文件才会被引入构建.源集合导致每个对象文件,被添加到最终二进制的链接行中.

- 如果你最终将代码链接到组件、共享库或可加载模块中, 则通常需要使用源集合.这是因为没有从共享库中引用的符号的对象文件,根本不会链接到最终库中.即使该对象文件有一个用于导出的标记符号,该标记符号的目标取决于共享库的需要,也会发生此遗漏. 这将导致链接以后的目标时,具有未定义符号的结果.

- 单元测试(以及任何带有副作用的静态初始化器)都必须使用源集合.这个 gtest TEST 宏创建注册测试的静态初始化器.但是,由于没有代码引用对象文件中的符号,因此将测试链接到静态库中,然后链接到测试可执行文件中, 这意味着测试将被剥离.

- 静态库由对象文件中的所有数据复制组成.这占用了更多的磁盘空间,对于大库中某些具有非常大对象文件的配置,可能会导致超出静态库大小的内部限制.源集合没有这个限制.根据构建配置,一些目标在源集合和静态库之间切换,以避免此问题.

- 源集合可以没有任何源,而静态库如果没有源代码,则会给出奇怪的特定平台错误.如果目标仅具有头信息(include检查目的)或 在sone 平台上若没有源,则使用源集.

- 如果某个链接不需要很多符号(在链接测试二进制文件时尤其如此),那么将该代码放入静态库可以显著提高链接性能.这是因为一开始就不考虑不需要链接的对象文件,而不是强制链接器在以后没有引用时,删除未使用的代码.

### 可加载模块与共享库与组件的比较

组件是来自 Chrome 的行话(而不是内置的 GN 概念),它扩展到共享库或静态库/源集合,具体取决于`is_component_build`变量.这允许发布构建大型二进制文件的中静态链接,但是对于开发人员来说,大多数操作都使用共享库.

共享库将列在依赖目标的链接行上,并在应用程序启动时由操作系统自动加载,符号自动解析.可加载模块不直接链接,应用程序必须手动加载它.

在 Windows 和 Linux 共享库和可加载模块中产生相同类型的文件(分别是`.dll`和`.so`).唯一的区别在于它们是如何与依赖的目标联系起来的.在这些平台上,有一个`deps`对可加载模块的依赖性与`data_deps`(非链接)依赖于共享库是一样的.

在 MAC 上,这些目标具有不同的格式:共享库将生成一个`.dylib`文件和可加载模块将生成一个`.so`文件.

使用插件之类的可加载模块.共享库应该很少在组件之外使用,因为大多数 Chrome 代码都是作为少量的大型二进制文件发送给最终用户的.在这类插件库的情况下,对于目标类型(甚至对于无关紧要的平台)使用可加载模块,和对于依赖于它的目标,使用数据仓库(data deps)都是很好的实践,因此从两个地方可以清楚地看到库将如何链接和加载.

## 构建参数

### 范围-Scope 

构建参数应该被范围化为一个行为单元,例如启用一个特征.通常,在导入文件中声明一个参数,以便共享它,让构建子集使用.

Chrome 有许多遗留标志参数`//build/config/features.gni`,`//build/config/ui.gni`. 这些位置被弃用. 特征标志参数应该与该功能的代码一起使用.如许多浏览器级功能可以在`//chrome/`某个地方找到,而没有底层代码.一些 UI 环境标志参数可以进入`//ui/`,还有许多标志参数的相应代码在`//components/`. 你可以写一个组件中的`.gni`文件,并在 Chrome 中有构建文件,或者在必要时导入内容.

思考事物的方式在`//build`目录,能将目录嵌入到各种项目中,如 V8 和 WebRTC. 特定外部的代码的构建标志参数不应该位于构建目录中,并且 V8 不应该获得 Chrome 功能的功能定义.

新的特征定义应该使用 Bu 建 dFLAG 系统.见`//build/buildflag_header.gni`它允许预处理器定义为模块化,而不需要过去使用全局定义的许多缺点.

### 类型

参数支持所有[GN 语言类型](language.zh.md#语言).

在绝大多数情况下`boolean`是首选类型,因为大多数参数是启用或禁用特征或includes.

`String`通常用于文件名.虽然它们也用于枚举类型,虽然`integer`有时也使用.

### 命名约定

虽然没有关于参数命名的硬性规则,但有许多共同的约定.如果您想查看当前使用的参数名和默认值的列表`gn args out/Debug --list --short`.

`use_foo`-指示依赖性或主要代码路径用于`include`(例如)`use_open_ssl`,`use_ozone`,`use_cups`)

`enable_foo`-指示要启用的特征或工具(例如).`enable_google_now`,`enable_nacl`,`enable_remoting`,`enable_pdf`)

`disable_foo` - *不*推荐,使用`enable_foo`替换默认值

`is_foo`-通常是全局状态描述符(例如`is_chrome_branded`,`is_desktop_linux`;非全球化的糟糕选择

`foo_use_bar`前缀可用于表示参数的有限范围(例如).`rtc_use_h264`,`v8_use_snapshot`)
