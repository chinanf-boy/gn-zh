# GN 语言与操作

<!-- [TOC] -->
<!-- START doctoc -->
<!-- END doctoc -->

## 介绍

本页描述了许多语言细节和行为.

### 使用内置帮助!

GN 有一个广泛的内置帮助系统,它为每个函数和内置变量提供了参考. 这个页面更高级.

```
gn help
```

你也可以看到[幻灯片](https://docs.google.com/presentation/d/15Zwb53JcncHfEwHpnG_PoIbbzQ3GQi_cpujYwbpcbZo/edit?usp=sharing)从 2016 年 3 月的 GN 介绍.讲演者笔记完整内容.

### 设计哲学

- 编写构建文件不应该是创造性的努力.理想情况下,两个人应该按照相同的要求生成相同的构建文件.除非绝对需要,否则不应该有任何灵活性.尽可能多的事情应该是致命的错误.

- 这个定义应该比规则更像代码.我不想编写或调试 Prolog.但是我们团队中的每个人都可以编写和调试 C++和 Python.

- 构建语言应该针对构建应该如何工作而自圆其说.它不一定容易或甚至可能表达任意的事物.我们应该改变源和工具以使构建更简单,而不是使所有事情更复杂以符合外部需求(在合理的范围内).

- 当它有意义的时候,要像火焰一样(参见下面的不同之处和相似之处).

## 语言

GN 使用非常简单、动态类型的语言.类型有:

- 布尔型(`true`,`false`)
- 64 位有符号整数.
- 串.
- 列表(任何其他类型).
- 范围(像字典一样,只用于内置的东西).

有些内置变量的值依赖于当前环境.见`gn help`更多.

语言中有许多故意遗漏的地方.例如,没有用户定义的函数调用(模板是最接近的).按照上面的设计理念,如果你需要这种东西,你可能做得不对.

变量`sources`有一个特殊的规则:在分配给它时,将一个排除模式列表应用于它.这是为了自动筛选出一些类型的文件.见`gn help set_sources_assignment_filter`和`gn help label_pattern`更多.

语言呆子的完整语法可在`gn help grammar`.

### 串

字符串以双引号括起来,并使用反斜杠作为转义字符.所支持的唯一转义序列为:

- `\"`(原文引用)
- `\$`(文字符号)
- `\\`(用于文字反斜杠)

反斜杠的任何其他用途都被视为字面反斜杠.因此,例如,`\b`在模式中使用不需要转义,大多数 Windows 路径也不需要`"C:\foo\bar.h"`.

支持简单变量替换`$`其中,跟随美元符号的词被变量的值替换.你可以随意地用这个名字包围这个名字.`{}`如果没有一个非变量名称字符来终止变量名.不支持更复杂的表达式,只有变量名替换.

```
a = "mypath"
b = "$a/foo.cc"  # b -> "mypath/foo.cc"
c = "foo${a}bar.cc"  # c -> "foomypathbar.cc"
```

您可以使用"$0xFF"语法编码 8 位字符,因此具有换行符(HEX 0A)的字符串将`"look$0x0Alike$0x0Athis"`.

### 列表

没有办法得到一个列表的长度.如果你发现自己想做这种事,你就要在建筑中做太多的工作.

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

列表上的运算符搜索匹配项并移除所有匹配项.从另一个列表中减去列表将删除第二个列表中的每个条目.

如果没有找到匹配的项,则会抛出一个错误,因此在删除项之前,您需要提前知道项在那里.由于无法测试是否包含,所以主要用例是设置文件或标志的主列表,并根据各种条件删除不适用于当前构建的文件或标志.

在风格上,只希望添加到列表中,并使每个源文件或依赖项出现一次.这与 Chrome 团队以前为 GYP 提供的建议相反(GYP 希望列出所有文件,然后删除条件中不需要的文件).

列表支持基于零的下标提取值:

```
a = [ "first", "second", "third" ]
b = a[1]  # -> "second"
```

这个\[]运算符是只读的,不能用于对列表进行突变.这种情况的主要用途是当外部脚本返回几个已知值时,您要提取它们.

有些情况下,当您要附加一个列表时,很容易覆盖它.为了帮助捕获此情况,将非空列表分配给包含现有非空列表的变量是错误的.如果您想绕过这个限制,首先将目标变量分配给空列表.

```
a = [ "one" ]
a = [ "two" ]  # Error: overwriting nonempty list with a nonempty list.
a = []         # OK
a = [ "two" ]  # OK
```

注意,构建脚本的执行没有对底层数据含义的内在了解.这意味着它不知道`sources`例如,是文件名的列表.因此,如果删除一个项,则它必须与文本字符串匹配,而不是指定将解析为相同文件名的不同名称.

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

您可以在列表中重复使用`foreach`. 这是灰心丧气的.构建应该做的大多数事情通常都可以在不这样做的情况下表达,并且如果您发现有必要,这可能表明您在元构建中做了太多的工作.

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

一些函数采用一个封闭的代码块.`{ }`跟随他们:

```
static_library("mylibrary") {
  sources = [ "a.cc" ]
}
```

这些都定义了目标.用户可以用下面讨论的模板机制定义新的函数.

确切地说,这个表达式意味着块成为函数执行的函数的参数.大多数块样式函数执行块,并将结果的范围视为变量读数字典.

### 范围与执行

文件和函数调用`{ }`块引入新的范围.作用域是嵌套的.当读取变量时,将以相反顺序搜索包含的范围,直到找到匹配的名称为止.变量写入总是到最内层范围.

除了最里面的一个以外,没有办法修改任何封闭范围.这意味着,例如,当您定义目标时,您在块内所做的任何操作都不会"泄漏"到文件的其余部分.

`if`/`else`/`foreach`语句,即使它们使用`{ }`不要引入新的范围,所以更改将持续到语句之外.

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

目标是生成图中的节点.它通常表示将生成的某种可执行文件或库文件.目标取决于其他目标.内置目标类型(参见`gn help <targettype>`更多的帮助是:

- `action`运行脚本生成文件.
- `action_foreach`为每个源文件运行一次脚本.
- `bundle_data`声明数据进入 MAC/iOS 包.
- `create_bundle`创建一个 Mac/IOS 包.
- `executable`:生成一个可执行文件.
- `group`是指一个或多个其他目标的虚拟依赖节点.
- `shared_library`A. DLL 或.SO.
- `loadable_module`A.DLL 或.SO 只能在运行时加载.
- `source_set`一个轻量级的虚拟静态库(通常比实际静态库更可取,因为它将更快地构建).
- `static_library`是.LIB 或.file(通常你需要一个`source_set`取而代之的是)

您可以使用模板来扩展自定义目标类型(见下文).在 Chrome 中,一些更常用的模板是:

- `component`无论是源集还是共享库,取决于构建类型.
- `test`一个可执行的测试.在移动中,这将为测试创建合适的本机应用程序类型.
- `app`:可执行文件或 MAC/IOS 应用程序.
- `android_apk`做一个 APK.有一个*许多*其他 Android 系统,请参见`//build/config/android/rules.gni`.

## 组态

SCONS 是指定标志集、目录和定义的命名对象.它们可以应用于目标并被推到相关目标.

定义配置:

```
config("myconfig") {
  includes = [ "src/include" ]
  defines = [ "ENABLE_DOOM_MELON" ]
}
```

将配置应用到目标:

```
executable("doom_melon") {
  configs = [ ":myconfig" ]
}
```

Buffic 配置文件通常指定设置默认配置列表的目标默认值.目标可以根据需要添加或删除该列表.所以在实践中你通常会用到`configs += ":myconfig"`追加到默认列表中.

见`gn help config`有关如何声明和应用配置的详细信息.

### 公共配置

目标可以将设置应用于依赖于它的其他目标.最常见的示例是第三方目标,它需要一些定义或包括目录才能正确编译其头部.您希望这些设置既适用于第三方库本身的编译,也适用于使用该库的所有目标.

要做到这一点,您要编写一个配置,您要应用的设置:

```
config("my_external_library_config") {
  includes = "."
  defines = [ "DISABLE_JANK" ]
}
```

然后将这个配置作为一个"公共"配置添加到目标中.它既适用于目标,也适用于直接依赖于它的目标.

```
shared_library("my_external_library") {
  ...
  # Targets that depend on this get this config applied.
  public_configs = [ ":my_external_library_config" ]
}
```

通过将目标添加为"公共"依赖项,依赖的目标可以反过来将此提升到依赖树的另一个层次.

```
static_library("intermediate_library") {
  ...
  # Targets that depend on this one also get the configs from "my external library".
  public_deps = [ ":my_external_library" ]
}
```

目标可以将配置转发到所有依赖者,直到将链接设置为一个链接边界为止.`all_dependent_config`. 这是非常令人沮丧的,因为它可以喷洒标志,并定义了更多的构建比必要的.相反,使用 Puffic ODEPS 来控制哪些标志应用于何处.

在 Chrome 中,更喜欢构建标志头系统.`build/buildflag_header.gni`用于定义编译器定义的大多数错误的定义.

## 模板

模板是 GN 重新使用代码的主要方式.通常,模板将扩展到一个或多个其他目标类型.

```
# Declares a script that compiles IDL files to source, and then compiles those
# source files.
template("idl") {
  # Always base helper targets on target_name so they're unique. Target name
  # will be the string passed as the name when the template is invoked.
  idl_target_name = "${target_name}_generate"
  action_foreach(idl_target_name) {
    ...
  }

  # Your template should always define a target with the name target_name.
  # When other targets depend on your template invocation, this will be the
  # destination of that dependency.
  source_set(target_name) {
    ...
    deps = [ ":$idl_target_name" ]  # Require the sources to be compiled.
  }
}
```

通常,模板定义将进入`.gni`文件和用户将导入该文件以查看模板定义:

```
import("//tools/idl_compiler.gni")

idl("my_interfaces") {
  sources = [ "a.idl", "b.idl" ]
}
```

声明模板在该范围内创建围绕范围内的变量的闭包.当调用模板时,神奇变量`invoker`用于从调用范围中读取变量.模板通常会将其感兴趣的值复制到它自己的范围内:

```
template("idl") {
  source_set(target_name) {
    sources = invoker.sources
  }
}
```

模板执行时的当前目录将是调用生成文件而不是模板源文件的当前目录.因此,从模板调用程序传入的文件将是正确的(这通常说明模板中的大多数文件处理).但是,如果模板本身具有文件(可能它生成运行脚本的操作),则您需要使用绝对路径("//foo/...")来引用这些文件,以说明当前目录在调用期间是不可预测的.见`gn help template`更多的信息和更完整的例子.

## 其他特征

### 进口

你可以进口`.gni`将文件放入当前范围`import`功能.这是*不*一个包含在 C++中的意义.导入的文件独立执行,并将所得的范围复制到当前文件中(C++在执行包含指令时的当前上下文中执行所包含的文件).这允许缓存导入的结果,并且还防止了一些"创造性的"使用 include(如多重包含的文件).

典型地,A`.gni`将定义生成参数和模板.见`gn help import`更多.

你的`.gni`文件可以通过使用前面的下划线来定义不包含导出文件的临时变量.`_this`.

### 路径处理

通常情况下,您希望生成一个文件名或与不同目录相对应的文件名列表.这在运行脚本时尤其常见,这些脚本以构建输出目录作为当前目录执行,而构建文件通常引用与其包含的目录相关的文件.

你可以使用`rebase_path`转换目录.见`gn help rebase_path`更多的帮助和例子.将文件名相对于当前目录转换为根目录的典型用法是:`new_paths = rebase_path("myfile.c", root_build_dir)`

### 模式

模式用于为自定义目标类型的给定输入集生成输出文件名,并从`sources`变量(见)`gn help set_sources_assignment_filter`)

它们就像简单的正则表达式.见`gn help label_pattern`更多.

### 执行脚本

执行脚本有两种方法.GN 中的所有外部脚本都在 Python 中.第一种方法是构建步骤.这样的脚本将输入一些输入,并生成一些输出作为构建的一部分.调用脚本的目标用"Actudio"目标类型声明(参见`gn help action`)

执行脚本的第二种方法在构建文件执行过程中是同步的.在某些情况下,这是必要的,以确定要编译的文件集,或者获得构建文件可能依赖的某些系统配置.生成文件可以读取脚本的 STDUT 并以不同的方式对其进行操作.

同步脚本执行由`exec_script`函数(见)`gn help exec_script`详情和例子).因为同步地执行脚本需要暂停当前构建文件的执行,直到 Python 进程完成执行,所以对外部脚本的依赖很慢,应该最小化.

为了防止滥用,允许调用的文件`exec_script`可以在白纸上白`.gn`文件.Chrome 这样做需要额外的代码审查这样的补充.见`gn help dotfile`.

可以同步读取和写入文件,这些文件在同步运行脚本时是不必要的,但偶尔也是必需的.典型的用例是传递比当前平台的命令行限制长的文件名列表.见`gn help read_file`和`gn help write_file`对于如何读取和写入文件.如果可能的话,应该避免这些功能.

超过命令行长度限制的操作可以使用响应文件绕过这个限制,而无需同步写入文件.见`gn help response_file_contents`.

# 火焰的异同

火焰是谷歌的内部构建系统,现在公开发布为[巴泽尔](http://bazel.io/). 它启发了许多其他系统,例如[裤子](http://www.pantsbuild.org/)和[巴克](http://facebook.github.io/buck/).

在谷歌的同构环境中,条件语句的需求非常低,它们可以通过几个黑客来完成.`abi_deps`)Chrome 使用的条件遍及各地,需要添加这些是文件看起来不同的主要原因.

GN 还添加了"配置"的概念,以管理服务器上同样不会出现的一些更棘手的依赖和配置问题.BRAZE 有一个"配置"的概念,它就像一个 GN 工具链,但内置于工具本身.工具链在 GN 中工作的方式是试图以干净的方式将这个概念分离到构建文件中的结果.

GN 保持一些 GYP 概念,如"所有依赖"设置,在火焰中有点不同.这部分是为了简化从现有 GYP 代码的转换,并且 GYP 结构通常提供更加细粒度的控制(根据情况,这是好还是坏).

GN 也使用 GYP 名称,比如"source"而不是"srcs",因为缩写看起来不必要地晦涩,尽管它使用 Blaze 的"deps",因为很难输入".ies".Chromium 还在一个目标中编译多种语言,因此删除了指定目标名称前缀上的语言类型(例如,从`cc_library`)
