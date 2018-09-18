
# GN快速入门指南

<!-- [TOC] -->

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [运行GN](#%E8%BF%90%E8%A1%8Cgn)
- [构建一个build](#%E6%9E%84%E5%BB%BA%E4%B8%80%E4%B8%AAbuild)
- [传递构建参数](#%E4%BC%A0%E9%80%92%E6%9E%84%E5%BB%BA%E5%8F%82%E6%95%B0)
- [跨平台编译到目标操作系统或体系结构](#%E8%B7%A8%E5%B9%B3%E5%8F%B0%E7%BC%96%E8%AF%91%E5%88%B0%E7%9B%AE%E6%A0%87%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%88%96%E4%BD%93%E7%B3%BB%E7%BB%93%E6%9E%84)
- [配置goma](#%E9%85%8D%E7%BD%AEgoma)
- [配置组件模式](#%E9%85%8D%E7%BD%AE%E7%BB%84%E4%BB%B6%E6%A8%A1%E5%BC%8F)
- [一步步](#%E4%B8%80%E6%AD%A5%E6%AD%A5)
  - [添加一个构建文件](#%E6%B7%BB%E5%8A%A0%E4%B8%80%E4%B8%AA%E6%9E%84%E5%BB%BA%E6%96%87%E4%BB%B6)
  - [测试您的添加](#%E6%B5%8B%E8%AF%95%E6%82%A8%E7%9A%84%E6%B7%BB%E5%8A%A0)
  - [声明依赖关系](#%E5%A3%B0%E6%98%8E%E4%BE%9D%E8%B5%96%E5%85%B3%E7%B3%BB)
  - [测试静态库版本](#%E6%B5%8B%E8%AF%95%E9%9D%99%E6%80%81%E5%BA%93%E7%89%88%E6%9C%AC)
  - [编译器设置](#%E7%BC%96%E8%AF%91%E5%99%A8%E8%AE%BE%E7%BD%AE)
  - [将设置放在配置中](#%E5%B0%86%E8%AE%BE%E7%BD%AE%E6%94%BE%E5%9C%A8%E9%85%8D%E7%BD%AE%E4%B8%AD)
  - [依赖配置](#%E4%BE%9D%E8%B5%96%E9%85%8D%E7%BD%AE)
- [添加新的构建参数](#%E6%B7%BB%E5%8A%A0%E6%96%B0%E7%9A%84%E6%9E%84%E5%BB%BA%E5%8F%82%E6%95%B0)
- [不知道发生了什么事?](#%E4%B8%8D%E7%9F%A5%E9%81%93%E5%8F%91%E7%94%9F%E4%BA%86%E4%BB%80%E4%B9%88%E4%BA%8B)
  - [打印调试](#%E6%89%93%E5%8D%B0%E8%B0%83%E8%AF%95)
  - ["desc"命令](#desc%E5%91%BD%E4%BB%A4)
  - [性能](#%E6%80%A7%E8%83%BD)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 运行GN

你可以在命令行里直接输入`gn`运行. 因为在`depot_tools`（路径应该在你的环境变量PATH中已经设置过）工具目录中有一个相同名字的脚本. 这个脚本会找到当前目录中的二进制文件并运行它. 

## 构建一个build

使用GYP时，系统会根据相应的配置参数分别生成`Debug`和`Release`编译目录. 但GN不一样，你可以任意配置你的编译参数和生成目录. 编译时如果检测到Ninja文件需要更新时，也会自动重新生成. 

要创建构建目录:

```
gn gen out/my_build
```

## 传递构建参数

通过运行以下命令在build目录中设置生成参数:

```
gn args out/my_build
```

这将启动一个编辑器. 在这个文件中键入构建参数，如下所示:

```
is_component_build = true
is_debug = false
```

您可以通过输入以下命令来查看可用参数列表及其默认值

```
gn args --list out/my_build
```

请注意，您必须指定此命令的构建目录，因为可用的参数可以根据设置的构建目录进行更改. 

Chrome开发人员还可以阅读[Chrome特定的的构建配置](http://www.chromium.org/developers/gn-build-configuration)说明以获取更多信息.

## 跨平台编译到目标操作系统或体系结构

运行`gn args out/Default`根据需要替换您的构建目录），并为常见的跨平台编译选项添加以下一行或多行. 

```
target_os = "chromeos"
target_os = "android"

target_cpu = "arm"
target_cpu = "x86"
target_cpu = "x64"
```

有关更多信息，请参阅[GNCrossCompiles](cross_compiles.zh.md).

## 配置goma

运行`gn args out/Default`(根据需要替换您的构建目录).加:

```
use_goma = true
goma_dir = "~/foo/bar/goma"
```

如果您的goma位于默认位置(`~/goma`), 那么你可以省略该goma_dir行. 

## 配置组件模式

这是一个像goma标志一样的构建参数.运行`gn args out/Default`并添加:

```
is_component_build = true
```

## 一步步

### 添加一个构建文件

创建一个`tools/gn/tutorial/BUILD.gn`文件并输入以下内容:

```
executable("hello_world") {
  sources = [
    "hello_world.cc",
  ]
}
```

该目录中应该已经有一个`hello_world.cc`文件，包含你所期望的. 就是这样！现在我们只需要告诉构建这个文件. 打开BUILD.gn根目录中的文件，并将此目标的标签添加到其中一个根组的依赖关系（"组"目标是一个元目标，它只是其他目标的集合）:

```
group("root") {
  deps = [
    ...
    "//url",
    "//tools/gn/tutorial:hello_world",
  ]
}
```

您可以看到您的目标标签是"//"（表示源根目录），后面是目录名称，冒号和目标名称. 

### 测试您的添加

从源根目录中打开命令行:

```
gn gen out/Default
ninja -C out/Default hello_world
out/Default/hello_world
```

GN鼓励目标名称不是全球唯一的静态库. 要构建其中之一，您可以将不带前导"//"的标签传递给Ninja:

```
ninja -C out/Default tools/gn/tutorial:hello_world
```

### 声明依赖关系

让我们来做一个静态库，它有一个函数向随机的人打招呼. 该目录中有一个源文件`hello.cc`，它具有执行此操作的功能. 打开`tools/gn/tutorial/BUILD.gn`文件并将静态库添加到现有文件的底部:

```
static_library("hello") {
  sources = [
    "hello.cc",
  ]
}
```

现在让我们添加一个依赖于这个库的可执行文件:

```
executable("say_hello") {
  sources = [
    "say_hello.cc",
  ]
  deps = [
    ":hello",
  ]
}
```

这个可执行文件包含一个源文件，并依赖于前一个静态库. 静态库在`deps`使用它的标签引用. 您可以使用完整的标签，`//tools/gn/tutorial:hello`但是如果您在同一个构建文件中引用目标，则可以使用快捷方式`:hello`. 

### 测试静态库版本

从源根目录中运行命令行:

```
ninja -C out/Default say_hello
out/Default/say_hello
```


请注意，您**不必**重新运行GN. 当任何构建文件发生变化时，GN将自动重建Ninja文件. 当Ninja在执行开始时打印[`1/1] Regenerating ninja files`时，就是这种情况. 

### 编译器设置

我们的hello 库有一个新功能，能够同时向两个人打招呼. 这个特性是通过定义`TWO_PEOPLE`来控制的. 我们可以像这样添加定义:

```
static_library("hello") {
  sources = [
    "hello.cc",
  ]
  defines = [
    "TWO_PEOPLE",
  ]
}
```

### 将设置放在配置中

然而，库的用户也需要知道这个定义，并把它放在静态库目标中，只为那里的文件定义它. 如果其他人包括`hello.h`，他们将不会看到新的定义. 要看到新的定义，每个人都必须定义`TWO_PEOPLE`. 

GN有一个叫"config"的概念封装设置. 我们来创建一个定义我们的预处理器的定义:

```
config("hello_config") {
  defines = [
    "TWO_PEOPLE",
  ]
}
```

要将这些设置应用于目标，只需要将配置标签添加到目标中的配置列表中即可:

```
static_library("hello") {
  ...
  configs += [
    ":hello_config",
  ]
}
```

请注意，您需要"+ ="而不是"="，因为构建配置具有应用于每个设置默认构建内容的目标的默认配置集. 你想添加到这个列表而不是覆盖它. 要查看默认配置，可以使用`print`构建文件中的函数或`desc`命令行子命令（请参阅下面的两个示例）. 

### 依赖配置

这很好地封装了我们的设置，但仍然需要每个使用我们库的用户来设置自己的配置. 如果依赖于我们的`hello`库的每个人都能自动获得这个信息，那就太好了. 将您的库定义更改为:

```
static_library("hello") {
  sources = [
    "hello.cc",
  ]
  all_dependent_configs = [
    ":hello_config"
  ]
}
```

这应用`hello_config`于`hello`目标本身，加上所有目标的传递依赖于目前的目标. 现在依赖我们的每个人都会得到我们的设置. 你也可以设置`public_configs`只适用于直接依赖你的目标（不是过渡性）. 

现在，如果您编译并运行，您将看到两个人的新版本:

```
> ninja -C out/Default say_hello
ninja: Entering directory 'out/Default'
[1/1] Regenerating ninja files
[4/4] LINK say_hello
> out/Default/say_hello
Hello, Bill and Joy.
```

## 添加新的构建参数

通过`declare_args`可以声明你接受哪些参数，并指定默认值. 

```
declare_args() {
  enable_teleporter = true
  enable_doom_melon = false
}
```

请参阅有关`gn help buildargs`如何工作的概述. 请参阅g`n help declare_args`声明它们的具体内容. 

在一个给定的范围内多次声明一个给定的参数是一个错误，因此应该在范围和命名参数中小心使用. 

## 不知道发生了什么事?

您可以在详细模式下运行GN，以查看有关正在执行的操作的许多消息. 使用`-v`. 

### 打印调试

有一个`print`命令仅写入标准输出:

```
static_library("hello") {
  ...
  print(configs)
}
```

这将打印适用于您的目标（包括默认的）的所有配置. 

### "desc"命令

您可以运行`gn desc <build_dir> <targetname>`以获取有关给定目标的信息:

```
gn desc out/Default //tools/gn/tutorial:say_hello
```

将打印出大量令人兴奋的信息. 您也可以只打印一个部分. 假设你想知道你的`TWO_PEOPLE`定义来自哪个`say_hello`目标:

```
> gn desc out/Default //tools/gn/tutorial:say_hello defines --blame
...lots of other stuff omitted...
  From //tools/gn/tutorial:hello_config
       (Added by //tools/gn/tutorial/BUILD.gn:12)
    TWO_PEOPLE
```

你可以看到，`TWO_PEOPLE`是由一个配置定义的，你也可以看到哪一行导致配置被应用到你的目标（在本例中，是`all_dependent_configs`行）. 

另一个特别有趣的变化

```
gn desc out/Default //base:base_i18n deps --tree
```

通过`gn help desc`了解更多信息. 

### 性能

通过使用`–time`命令行标志运行它，可以看到花了多长时间. 这将输出各种事物的时间统计. 

您还可以跟踪构建文件的执行方式:

```
gn --tracelog=mylog.trace
```

并且您可以在Chrome的`about:tracing`页面中加载生成的文件来查看所有内容. 
