# GN 语言与操作

<!-- [TOC] -->
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


  - [介绍](#%E4%BB%8B%E7%BB%8D)
    - [使用内置帮助!](#%E4%BD%BF%E7%94%A8%E5%86%85%E7%BD%AE%E5%B8%AE%E5%8A%A9)
    - [设计哲学](#%E8%AE%BE%E8%AE%A1%E5%93%B2%E5%AD%A6)
  - [语言](#%E8%AF%AD%E8%A8%80)
    - [字符串](#%E5%AD%97%E7%AC%A6%E4%B8%B2)
    - [列表](#%E5%88%97%E8%A1%A8)
    - [条件句](#%E6%9D%A1%E4%BB%B6%E5%8F%A5)
    - [循环](#%E5%BE%AA%E7%8E%AF)
    - [函数调用](#%E5%87%BD%E6%95%B0%E8%B0%83%E7%94%A8)
    - [范围与执行](#%E8%8C%83%E5%9B%B4%E4%B8%8E%E6%89%A7%E8%A1%8C)
  - [命名事物](#%E5%91%BD%E5%90%8D%E4%BA%8B%E7%89%A9)
    - [文件名和目录名](#%E6%96%87%E4%BB%B6%E5%90%8D%E5%92%8C%E7%9B%AE%E5%BD%95%E5%90%8D)
  - [构建配置](#%E6%9E%84%E5%BB%BA%E9%85%8D%E7%BD%AE)
  - [目标](#%E7%9B%AE%E6%A0%87)
  - [Configs](#configs)
    - [公共配置](#%E5%85%AC%E5%85%B1%E9%85%8D%E7%BD%AE)
  - [模板](#%E6%A8%A1%E6%9D%BF)
  - [其他特征](#%E5%85%B6%E4%BB%96%E7%89%B9%E5%BE%81)
    - [导入](#%E5%AF%BC%E5%85%A5)
    - [路径处理](#%E8%B7%AF%E5%BE%84%E5%A4%84%E7%90%86)
    - [模式](#%E6%A8%A1%E5%BC%8F)
    - [执行脚本](#%E6%89%A7%E8%A1%8C%E8%84%9A%E6%9C%AC)
- [Blaze的异同](#blaze%E7%9A%84%E5%BC%82%E5%90%8C)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 介绍

本页描述了许多语言细节和行为.

### 使用内置帮助!

GN 有一个广泛的内置帮助系统,它为每个函数和内置变量提供了参考. 这个页面更高级.

```
gn help
```

你也可以看到[幻灯片](https://docs.google.com/presentation/d/15Zwb53JcncHfEwHpnG_PoIbbzQ3GQi_cpujYwbpcbZo/edit?usp=sharing)从 2016 年 3 月的 GN 介绍.讲演者笔记完整内容.

### 设计哲学

- 编写构建文件不应该是创造的事情.理想情况下,两个人应该按照相同的要求生成相同的构建文件.除非绝对需要,否则不应该有任何灵活性. 尽可能多的事情应该是致命的错误.

- 这些定义应该比规则更像代码.我不想编写或调试 Prolog. 但是我们团队中的每个人都可以编写和调试 C++和 Python.

- 构建语言应该针对构建应该如何工作而自圆其说. 它不一定容易或甚至可能表达任意的事物.我们应该改变 源 和 工具 以使构建更简单,而不是使所有事情更复杂以符合外部需求(在合理的范围内).

- 当它有意义的时候,要像Blaze一样(参见下面的不同之处和相似之处).

> 没看懂, 不看也行

## 语言

GN 使用非常简单、动态类型的语言.类型有:

- 布尔型(`true`,`false`)
- 64 位有符号整数.
- string.
- 列表(任何其他类型).
- 范围(像字典一样,只用于内置的东西).

有些内置变量的值依赖于当前环境. 参见`gn help`更多.

语言中有许多故意遗漏的地方.例如,没有用户定义的函数调用(模板是最接近的).按照上面的设计理念,如果你需要这种东西,你可能做得不对.

变量`sources`有一个特殊的规则: 在分配给它时,将一个排除的模式列表应用于它.这是为了自动筛选出一些类型的文件.参见`gn help set_sources_assignment_filter`和`gn help label_pattern`更多.

语言呆子, 可看完整语法在`gn help grammar`.

### 字符串

字符串以双引号括起来,并使用反斜杠作为转义字符. 所支持的唯一转义序列为:

- `\"`(原文引用)
- `\$`(文字符号)
- `\\`(用于文字反斜杠)

反斜杠的任何其他用途都被视为字面反斜杠.因此,例如,`\b`在模式中使用不需要转义,大多数 Windows 路径也不需要`"C:\foo\bar.h"`.

支持简单变量`$`,跟随美元符号的词被变量的值替换.你可以随意地用这个`{}`,将一个变量名来替换变量值.不支持更复杂的表达式,只有变量名替换.

```
a = "mypath"
b = "$a/foo.cc"  # b -> "mypath/foo.cc"
c = "foo${a}bar.cc"  # c -> "foomypathbar.cc"
```

您可以使用"$0xFF"语法编码 8 位字符,因此具有换行符(HEX 0A)的字符串会是`"look$0x0Alike$0x0Athis"`.

### 列表

没有办法得到一个列表的`长度`.如果你发现自己想做这种事,你就要在建筑中做太多的工作.

列表支持附加:

```
a = [ "first" ]
a += [ "second" ]  # [ "first", "second" ]
a += [ "third", "fourth" ]  # [ "first", "second", "third", "fourth" ]
b = a + [ "fifth" ]  # [ "first", "second", "third", "fourth", "fifth" ]
```

将列表追加到另一个列表中,将项目追加到第二个列表中,而不是将列表作为嵌套成员添加.

您可以从列表中删除项目:

```
a = [ "first", "second", "third", "first" ]
b = a - [ "first" ]  # [ "second", "third" ]
a -= [ "second" ]  # [ "first", "third", "fourth" ]
```

列表上的运算符搜索匹配项,并移除所有匹配项.从另一个列表中减去列表将删除列表中的每个 second 条目.

如果没有找到匹配的项,则会抛出一个错误,因此在删除项之前,您需要提前知道项在那里.由于无法测试是否包含,所以主要用例是设置文件或参数的主列表,并根据各种条件删除不适用于当前构建的文件或参数.

在风格上,只能添加到列表中,并使每个源文件或依赖项出现一次.这与 Chrome 团队以前为 GYP 提供的建议相反 (GYP 希望列出所有文件,然后删除条件中不需要的文件).

列表支持基于零的下标提取值:

```
a = [ "first", "second", "third" ]
b = a[1]  # -> "second"
```

这个\[]运算符是只读的,不能用于对列表进行改变.这种情况的主要用途是当外部脚本返回几个已知值时,您要提取它们.

有些情况下,当您要附加一个列表时,很容易覆盖它.为了帮助捕获此情况,将非空列表分配给 -> 现有包含非空列表的变量是错误的.如果您想绕过这个限制,首先将目标变量分配给空列表.

```
a = [ "one" ]
a = [ "two" ]  # Error: overwriting nonempty list with a nonempty list.
a = []         # OK
a = [ "two" ]  # OK
```

注意,构建脚本的执行, 没有对底层数据含义进行深入了解.这意味着,例如它不知道`sources`,是文件名的列表.因此,如果删除一个项,则它必须与文本字符串匹配,而不是指定,就能将同文件名解析为不同名称.

### 条件句

条件句看起来像 C:

```
  if (is_linux || (is_win && target_cpu == "x86")) {
    sources -= [ "something.cc" ]
  } else if (...) {
    ...
  } else {
    ...
  }
```

如果目标只在某些情况下声明,则可以在大多数地方使用,甚至在整个目标周围使用.

### 循环

您可以`foreach`列表,重复使用 这是灰心丧气的. 构建应该做的大多数事情通常都可以在不这样做的情况下表达,并且如果您发现有必要,这可能表明您在元构建中做了太多的工作.

```
foreach(i, mylist) {
  print(i)  # Note: i is a copy of each element, not a reference to it.
}
```

### 函数调用

简单函数调用看起来像大多数其他语言:

```
print("hello, world")
assert(is_win, "This should only be executed on Windows")
```

这样的功能是内置的,用户不能定义新的.

一些函数采用一个封闭的代码块`{ }`跟着他们:

```
static_library("mylibrary") {
  sources = [ "a.cc" ]
}
```

这些都定义了目标. 用户可以用下面讨论的模板机制定义新的函数.

确切地说,这个表达式意味着,区块成为函数执行的函数的参数.大多数区块样式函数会执行区块,并将结果的范围视为字典变量来读.

### 范围与执行

文件和函数调用`{ }`区块出现新的范围. 作用域是嵌套的.当读取变量时,将以相反顺序(内部找外部)搜索包含的范围,直到找到匹配的名称为止.变量的写入声明总是可以到最内层范围.

除了最里面的一个以外,没有办法修改任何闭包范围.这意味着,例如,当您定义目标时,您在区块内所做的任何操作都不会"泄漏"到文件的其余部分.

而`if`/`else`/`foreach`语句,即使它们使用`{ }`,并没出现新的范围,所以更改将持续到语句之外.

## 命名事物

### 文件名和目录名

文件和目录名是字符串,并被解释为相对于当前生成文件的目录.有三种可能的形式:

相对名称:

```
"foo.cc"
"src/foo.cc"
"../src/foo.cc"
```

源树绝对名称:

```
"//net/foo.cc"
"//base/test/foo.cc"
```

系统绝对名称(罕见的,通常用于包含目录):

```
"/usr/local/include/"
"/C:/Program Files/Windows Kits/Include"
```

## 构建配置

## 目标

目标是生成图中的节点.它通常表示将要生成的某种可执行文件或库文件.目标取决于其他目标. 内置目标类型(参见`gn help <targettype>`更多的帮助是:

- `action`运行脚本生成文件.
- `action_foreach`为每个源文件运行一次脚本.
- `bundle_dat一个`声明数据进入 MAC/iOS 包.
- `create_bundle`创建一个 Mac/IOS 包.
- `executable`:生成一个可执行文件.
- `group`是指一个或多个其他目标的虚拟依赖节点.
- `shared_library`一个 . DLL 或.SO.
- `loadable_module`一个 .DLL 或.SO ,只能在运行时加载.
- `source_set`一个轻量级的虚拟静态库(通常比实际静态库更可取,因为它将更快地构建).
- `static_library`是.LIB 或.file(通常你需要一个`source_set`取而代之)

您可以使用模板来扩展自定义目标类型(见下文).在 Chrome 中,一些更常用的模板是:

- `component`无论是源集和还是共享库,取决于什么构建类型.
- `test`一个可执行的测试.在移动端中,这将为测试创建合适的本机应用程序类型.
- `app`:可执行文件或 MAC/IOS 应用程序.
- `android_apk`做一个 APK.有一个*许多*其他 Android 系统,请参见`//build/config/android/rules.gni`.

## Configs

Configs 是指定参数集、目录和定义的命名对象.它们可以应用于目标并被推到相关目标.

定义配置:

``` bash
config("myconfig") {
  includes = [ "src/include" ]
  defines = [ "ENABLE_DOOM_MELON" ]
}
```

将配置应用到目标:

``` bash
executable("doom_melon") {
  configs = [ ":myconfig" ]
}
```

通常,这帮构建配置文件设置默认配置列表的目标默认值.目标可以根据需要添加或删除该列表.所以在实践中你通常会用到`configs += ":myconfig"`追加到默认列表中.

参见`gn help config`有关如何声明和应用配置的详细信息.

### 公共配置

目标可以将设置应用于依赖于它的其他目标.最常见的示例是第三方目标,它需要一些定义或包括目录才能正确编译其头部.您希望这些设置既适用于第三方库本身的编译,也适用于使用该库的所有目标.

要做到这一点,您要编写一个配置,您要应用的设置:

``` bash
config("my_external_library_config") {
  includes = "."
  defines = [ "DISABLE_JANK" ]
}
```

然后将这个配置作为一个"公共"配置添加到目标中.它既适用于目标,也适用于直接依赖于它的目标.

```bash
shared_library("my_external_library") {
  ...
  # 所有依赖于这个库的目标都拿到了公共配置.
  public_configs = [ ":my_external_library_config" ]
}
```

通过将目标添加为"公共"依赖项,依赖的目标可以反过来将此提升到依赖树的另一个层次.

```bash
static_library("intermediate_library") {
  ...
  # Targets that depend on this one also get the configs from "my external library".
  public_deps = [ ":my_external_library" ]
}
```

目标可以将配置转发到所有依赖者,直到将链接设置`all_dependent_config`为一个链接边界为止. 这是非常令人沮丧的,因为它可以喷洒标志,并定义了比所需的更多构建.相反,我们可以使用 public_deps 来控制哪些标志参数应用于何处.

在 Chrome 中,更喜欢构建标志参数头系统.`build/buildflag_header.gni`用于定义编译器定义的大多数错误的定义.

## 模板

Templates 是 GN 复用代码的主要方式.通常,模板扩展为一个或多个其他目标类型.

```bash
# 声明一个脚本，将IDL文件编译为源，然后编译它们
# 源文件.
template("idl") {
  # 始终在target_name上创建辅助目标，因此它们是唯一的.
  # target_name将是调用模板时,作为名称传递的字符串。
  idl_target_name = "${target_name}_generate"
  action_foreach(idl_target_name) {
    ...
  }

  # 您的模板应始终定义名为target_name的目标。
  # 当其他目标依赖于您的模板调用时，这将是
  # 对应target_name依赖的目的地。
  source_set(target_name) {
    ...
    deps = [ ":$idl_target_name" ]  # 要求编译源代码。
  }
}
```

通常,模板定义放在`.gni`文件和用户导入该文件以使用定义的模板:

```r
import("//tools/idl_compiler.gni")

idl("my_interfaces") {
  sources = [ "a.idl", "b.idl" ]
}
```

声明模板的创建带有变量的`{}`闭包. 当调用模板时,神奇变量`invoker`用于从调用定义中读取变量. 模板通常会将其感兴趣的值复制到它自己的范围内:

```
template("idl") {
  source_set(target_name) {
    sources = invoker.sources
  }
}
```

模板执行时的当前目录,会是调用生成文件,而不是模板源文件的当前目录.因此,从模板`invoker`传入的文件将是正确的(这通常说明模板中的大多数文件处理).但是,如果模板本身具有文件(可能它生成运行脚本的操作),则您需要使用绝对路径("//foo/...")来引用这些文件,以此规避当前目录在调用期间是不可预测性. 参见`gn help template`更多的信息和更完整的例子.

## 其他特征

### 导入

你可以通过`import`函数导入`.gni`文件,放入当前范围.这*不是*一个在 C++中的`include`意义. 导入的文件独立执行,并将所得的范围复制到当前文件中(C++在执行`include`指令时的当前上下文中,执行所包含的文件). 这允许缓存导入的结果,并且还防止了一些"创造性的"使用 include如包含多文件.

典型地,一个`.gni`将定义生成参数和模板,阅览`gn help import`获得更多.

你的`.gni`文件可以通过使用前面的下划线`_this`,来定义导出文件不被包含的临时变量.

### 路径处理

通常情况下,您希望生成一个文件名或与对应不同目录的文件名列表.这在运行脚本时尤其常见,这些脚本以构建输出目录作为当前目录执行,而构建文件通常是相对引用于自身目录.

你可以使用`rebase_path`转换目录.参见`gn help rebase_path`更多的帮助和例子.将相对于当前目录的文件名转换为根目录的典型用法是:`new_paths = rebase_path("myfile.c", root_build_dir)`

### 模式

模式用于为自定义目标类型的输入集合,生成输出文件名,并且必然从`sources`变量移除文件 (查阅`gn help set_sources_assignment_filter`)

它们就像简单的正则表达式.参见`gn help label_pattern`更多.

### 执行脚本

执行脚本有两种方法.GN 中的所有外部脚本都在 Python 中.第一种方法是构建步骤.这样的脚本将输入一些输入,并生成一些输出作为构建的一部分. 这样的调用脚本使用"action"作为目标类型声明(参见`gn help action`)

执行脚本的第二种方法在构建文件执行过程中是同步的.在某些情况下,为了确定编译的文件集,这是必要的,或者获得构建文件可能依赖的某些系统配置. 生成文件可以读取脚本的 stdout, 并以不同的方式对其进行操作.

同步脚本执行由`exec_script`函数提供(参考)`gn help exec_script`详情和例子).因为同步地执行脚本需要 *暂停* 当前构建文件的执行,直到 Python 进程完成执行,所以对外部脚本的依赖很慢,应该确保最小化.

为了防止滥用,允许调用`exec_script`的文件需要在最高级的`.gn`文件的白名单上. Chrome 要求对这样的补充,进行额外的代码审查.参见`gn help dotfile`.

可以同步读取和写入文件,这些文件在同步运行脚本时是不必要的,但偶尔也是必需的.典型的用例是传递,比当前平台的命令行限制长的文件名列表.见`gn help read_file`和`gn help write_file`对于如何读取和写入文件.如果可能的话,应该避免使用这些函数.

超过命令行长度限制的操作,可以使用响应文件绕过这个限制,而无需同步写入文件.参见`gn help response_file_contents`.

# Blaze的异同

Blaze是谷歌的内部构建系统,现在公开发布为[bazel](http://bazel.io/). 它启发了许多其他系统,例如[Pants](http://www.pantsbuild.org/)和[Buck](http://facebook.github.io/buck/).

在谷歌的同构环境中,条件语句的需求非常低,它们可以通过几个黑客手段(`abi_deps`)来完成.Chrome 使用的条件语法到处都是,而添加这些的主要原因是文件要看起来不同.

GN 还添加了"configs"的概念,以管理服务器上不会出现的同样一些更棘手的依赖和配置问题.bazel 有一个"configuration"的概念,它就像一个 GN toolchain,但内置于工具本身.工具链在 GN 中工作的方式是试图以干净的方式将这个概念从构建文件中分离开的结果.

GN 保持一些 GYP 概念,如"所有依赖"设置,在Blaze中有点不同.这部分是为了简化,是从现有 GYP 代码的转换来的,并且 GYP 结构通常提供更加细粒度的控制(根据情况,这是好还是坏).

GN 也使用 GYP 名称,比如"source"而不是"srcs",因为缩写看起来是那么晦涩,尽管它使用 Blaze 的"deps",也是因为"dependencies"很难输入.Chromium 还在一个目标中编译多种语言,因此删除了指定语言类型的目标名称前缀(例如,从`cc_library`)
