
# GN如何处理交叉编译

## 作为GN用户

GN为在单个构建中进行交叉编译和构建多个体系结构的东西提供了强大的支持(例如,构建一些在本地运行的东西以及一些在嵌入式设备上运行的东西).实际上,您可以同时构建的不同体系结构的数量没有限制;Chromium构建在某些配置中至少使用四个.

首先,GN有一个概念*主办*和a*目标*.主机是运行构建的平台,目标是代码实际运行的平台(这与...不同)[自动工具](http://www.gnu.org/software/automake/manual/html_node/Cross_002dCompilation.html)'术语,但使用更常见的术语进行交叉编译**).**

(令人困惑的是,GN还指每个构建工件 - 可执行文件,库等 - 作为目标.在此页面上,我们将仅使用"target"来引用您要运行代码的系统,并且使用"rule"或其他同义词来指代特定的构建工件.

当GN启动时,`host_os`和`host_cpu`自动设置变量以匹配操作系统(它们可以在args文件中重写,这在奇怪的角落情况下很有用).用户可以通过设置其中一个或两个来指定他们想要进行交叉编译`target_os`和`target_cpu`;如果它们没有设置,构建配置文件通常会将它们设置为主机的值,尽管Chromium构建将设置目标\_cpu如果目标"武装"\_os设置为"android").

因此,例如,在x64 Linux机器上运行:

```
gn gen out/Default
```

相当于:

```
gn gen out/Default --args='target_os="linux" target_cpu="x64"'
```

要进行32位ARM Android交叉编译,请执行以下操作:

```
gn gen out/Default --args='target_os="android"'
```

(我们不必指定目标\_cpu因为上面提到的条件).

并且,要进行64位MIPS Chrome OS交叉编译:

```
gn gen out/Default --args='target_os="chromeos" target_cpu="mips64el"'
```

## 作为BUILD.gn的作者

如果您正在编辑// build目录之外的构建文件(即,不直接处理工具链,编译器配置等),通常只需要担心一些事情:

该`current_toolchain`,`current_cpu`,和`current_os`变量反映了设置**目前**在给定的规则中有效.该`is_linux`,`is_win`等变量更新以反映当前设置,并更改为`cflags`,`ldflags`等等也只适用于当前的工具链和当前正在构建的东西.

你也可以参考`target_cpu`和`target_os`变量.如果您需要在主机上执行不同的操作(取决于哪个目标),这将非常有用\_要求拱;所有工具链中的值都是不变的.你可以为此做类似的事情`host_cpu`和`host_os`变量,但通常应该永远不需要.

对于默认工具链,`target_cpu`和`current_cpu`是相同的.对于辅助工具链,`current_cpu`基于工具链定义和设置`target_cpu`保持不变.在编写规则时,**`current_cpu`应该使用而不是`target_cpu`大多数时候**.

默认情况下,依赖项列在`deps`规则的变量使用相同的(当前活动的)工具链.您可以使用指定不同的工具链`foo(bar)`标签符号,如[参考文档的标签部分](reference.md#Toolchains).

这是一个何时使用的示例`target_cpu`vs`current_cpu`:

```
declare_args() {
  # Applies only to toolchains targeting target_cpu.
  sysroot = ""
}

config("my_config") {
  # Uses current_cpu because compile flags are toolchain-dependent.
  if (current_cpu == "arm") {
    defines = [ "CPU_IS_32_BIT" ]
  } else {
    defines = [ "CPU_IS_64_BIT" ]
  }
  # Compares current_cpu with target_cpu to see whether current_toolchain
  # has the same architecture as target_toolchain.
  if (sysroot != "" && current_cpu == target_cpu) {
    cflags = [
      "-isysroot",
      sysroot,
    ]
  }
}
```

## 作为// build / config或// build / toolchain作者

该`default_toolchain`宣告在`//build/config/BUILDCONFIG.gn`文件.通常是`default_toolchain`应该是工具链`target_os`和`target_cpu`.该`current_toolchain`反映了当前对规则有效的工具链.

一定要了解它们之间的区别`host_cpu`,`target_cpu`,`current_cpu`,和`toolchain_cpu`(和os等价物).前两个如上所述设置.你有责任确保这一点`current_cpu`在工具链定义中正确设置;如果您使用的是股票模板`gcc_toolchain`和`msvc_toolchain`,这意味着你有责任确保这一点`toolchain_cpu`和`toolchain_os`在模板调用中适当设置.
