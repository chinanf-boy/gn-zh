
# GN快速入门指南

[TOC]

## 运行GN

你跑了`gn`从命令行.有一个脚本`depot_tools`,大概在你的PATH中,有这个名字.该脚本将在源树中找到包含当前目录的二进制文件并运行它.

## 设置构建

在GYP中,系统会生成`Debug`和`Release`为您构建目录并相应地配置它们.GN没有这样做.相反,您可以使用所需的任何配置设置所需的任何构建目录.如果在构建该目录时它们已过期,Ninja文件将自动重新生成.

要创建构建目录:

```
gn gen out/my_build
```

## 传递构建参数

通过运行以下命令在构建目录中设置构建参数

```
gn args out/my_build
```

这将带来一个编辑.在这个文件中键入build args,如下所示:

```
is_component_build = true
is_debug = false
```

您可以通过键入来查看可用参数列表及其默认值

```
gn args --list out/my_build
```

在命令行上.请注意,您必须为此命令指定构建目录,因为可用参数可以根据设置进行更改.

Chrome开发人员也可以阅读[特定于Chrome的构建配置](http://www.chromium.org/developers/gn-build-configuration)有关更多信息的说明.

## 交叉编译到目标操作系统或体系结构

跑`gn args out/Default`(根据需要替换您的构建目录)并为常见的交叉编译选项添加以下一行或多行.

```
target_os = "chromeos"
target_os = "android"

target_cpu = "arm"
target_cpu = "x86"
target_cpu = "x64"
```

看到[GNCrossCompiles](cross_compiles.md)了解更多信息.

## 配置goma

跑`gn args out/Default`(根据需要替换您的构建目录).加:

```
use_goma = true
goma_dir = "~/foo/bar/goma"
```

如果您的goma位于默认位置(`~/goma`)然后你可以省略`goma_dir`线.

## 配置组件模式

这是一个像goma标志一样的构建arg.跑`gn args out/Default`并添加:

```
is_component_build = true
```

## 一步步

### 添加构建文件

创建一个`tools/gn/tutorial/BUILD.gn`文件并输入以下内容:

```
executable("hello_world") {
  sources = [
    "hello_world.cc",
  ]
}
```

应该已经有了`hello_world.cc`该目录中的文件,包含您期望的内容.而已!现在我们只需要告诉构建这个文件.打开`BUILD.gn`将文件放在根目录中,并将此目标的标签添加到其中一个根组的依赖项中("组"目标是一个元目标,它只是其他目标的集合):

```
group("root") {
  deps = [
    ...
    "//url",
    "//tools/gn/tutorial:hello_world",
  ]
}
```

您可以看到目标的标签是"//"(表示源根目录),后跟目录名称,冒号和目标名称.

### 测试您的添加

从源根目录中的命令行:

```
gn gen out/Default
ninja -C out/Default hello_world
out/Default/hello_world
```

GN鼓励非全局唯一的静态库的目标名称.要构建其中一个,您可以将没有前导"//"的标签传递给ninja:

```
ninja -C out/Default tools/gn/tutorial:hello_world
```

### 声明依赖项

让我们创建一个静态库,它具有向随机人员问好的功能.有一个源文件`hello.cc`在该目录中具有执行此操作的功能.打开`tools/gn/tutorial/BUILD.gn`文件并将静态库添加到现有文件的底部:

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

此可执行文件包含一个源文件,并依赖于以前的静态库.静态库由其中的标签引用`deps`.您可以使用完整标签`//tools/gn/tutorial:hello`但是如果您在同一个构建文件中引用目标,则可以使用该快捷方式`:hello`.

### 测试静态库版本

从源根目录中的命令行:

```
ninja -C out/Default say_hello
out/Default/say_hello
```

请注意你**没有**需要重新运行GN.当任何构建文件发生更改时,GN将自动重建ninja文件.你知道当ninja打印时会发生这种情况`[1/1] Regenerating ninja files`在执行开始时.

### 编译器设置

我们的hello库有一个新功能,可以同时向两个人打招呼.此功能由定义控制`TWO_PEOPLE`.我们可以像这样添加定义:

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

但是,库的用户还需要了解此定义,并将其放在静态库目标中,仅为其中的文件定义它.如果其他人包括`hello.h`,他们不会看到新的定义.要查看新定义,每个人都必须定义`TWO_PEOPLE`.

GN有一个名为"config"的概念,它封装了设置.让我们创建一个定义我们的预处理器定义:

```
config("hello_config") {
  defines = [
    "TWO_PEOPLE",
  ]
}
```

要将这些设置应用于目标,您只需将配置标签添加到目标中的配置列表中:

```
static_library("hello") {
  ...
  configs += [
    ":hello_config",
  ]
}
```

请注意,此处需要"+ ="而不是"=",因为构建配置具有应用于设置默认构建内容的每个目标的默认配置集.您想要添加到此列表而不是覆盖它.要查看默认配置,您可以使用`print`在构建文件中的函数或`desc`命令行子命令(参见下面的两个示例).

### 依赖配置

这很好地封装了我们的设置,但仍然需要使用我们库的每个人自己设置配置.如果每个人都依赖我们,这将是很好的`hello`库可以自动获得.将库定义更改为:

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

这适用于`hello_config`到了`hello`目标本身,以及所有过渡依赖于当前目标的目标.现在每个依赖我们的人都会得到我们的设置.你也可以设置`public_configs`它仅适用于直接依赖于目标的目标(不是传递性的).

现在,如果您编译并运行,您将看到有两个人的新版本:

```
> ninja -C out/Default say_hello
ninja: Entering directory 'out/Default'
[1/1] Regenerating ninja files
[4/4] LINK say_hello
> out/Default/say_hello
Hello, Bill and Joy.
```

## 添加新的构建参数

您声明接受哪些参数并通过指定默认值`declare_args`.

```
declare_args() {
  enable_teleporter = true
  enable_doom_melon = false
}
```

看到`gn help buildargs`了解其工作原理.看到`gn help declare_args`有关声明它们的细节.

在给定范围内多次声明给定参数是错误的,因此应该在范围界定和命名参数中使用.

## 不知道发生了什么事?

您可以在详细模式下运行GN以查看有关其正在执行的操作的大量消息.使用`-v`为了这.

### 打印调试

有一个`print`刚刚写入stdout的命令:

```
static_library("hello") {
  ...
  print(configs)
}
```

这将打印适用于您的目标的所有配置(包括默认配置).

### "desc"命令

你可以跑`gn desc <build_dir> <targetname>`获取有关给定目标的信息:

```
gn desc out/Default //tools/gn/tutorial:say_hello
```

将打印出许多令人兴奋的信息.您也可以只打印一个部分.让我们说你想知道你的位置`TWO_PEOPLE`定义来自于`say_hello`目标:

```
> gn desc out/Default //tools/gn/tutorial:say_hello defines --blame
...lots of other stuff omitted...
  From //tools/gn/tutorial:hello_config
       (Added by //tools/gn/tutorial/BUILD.gn:12)
    TWO_PEOPLE
```

你可以看到`TWO_PEOPLE`由配置定义,您还可以看到哪一行导致该配置应用于您的目标(在本例中为`all_dependent_configs`线).

另一个特别有趣的变化

```
gn desc out/Default //base:base_i18n deps --tree
```

看到`gn help desc`更多.

### 性能

您可以通过--time命令行标志运行它来查看花了很长时间.这将输出各种事物的计时摘要.

您还可以跟踪构建文件的执行方式:

```
gn --tracelog=mylog.trace
```

并且您可以在Chrome中加载生成的文件`about:tracing`页面看一切.
