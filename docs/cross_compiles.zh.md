
# GN如何处理 跨平台编译

## 作为GN用户

GN为你在单个构建中进行跨平台编译和构建多个体系结构的东西提供了强大的支持 ( 例如,构建一些在本地运行的东西,以及一些在嵌入式设备上运行的东西).实际上,您可以没有限制数量的同时构建不同体系结构;Chromium构建在某些配置中至少使用四个不同结构.

首先,GN有一个*host*和一个*target*的概念.主机是运行构建的平台,目标是代码实际运行的平台(这与[autotools-自动工具](http://www.gnu.org/software/automake/manual/html_node/Cross_002dCompilation.html)'术语不同,但使用更常见的术语 - **跨平台编译「cross-compiling」*).

(令人困惑的是,GN还指定构建时每个工件 - 可执行文件,库 等, 也可作为目标.在此页面上,我们将仅使用"target"来表明您要运行代码的系统,并且使用"rule"或其他同义词来代指特定的构建工件.

当GN启动时,`host_os`和`host_cpu`自动设置变量以匹配操作系统(它们可以在args文件中重写,这在奇怪的角落情况下很有用).用户可以通过设置其中`target_os`和`target_cpu`一个或两个来指定他们想要进行的跨平台编译; 如果它们没有设置,构建配置文件通常会将它们设置为主机的值,但Chromium构建会设置`target_cpu`为"arm", 如果`target_os`是设为"android"的话).

因此,例如,在x64 Linux机器上运行:

```
gn gen out/Default
```

相当于:

```
gn gen out/Default --args='target_os="linux" target_cpu="x64"'
```

要进行32位ARM Android 跨平台编译,请执行以下操作:

```
gn gen out/Default --args='target_os="android"'
```

(我们不必指定`target_cpu`因为上面提到的条件).

并且,要进行64位MIPS Chrome OS 跨平台编译:

```
gn gen out/Default --args='target_os="chromeos" target_cpu="mips64el"'
```

## 作为BUILD.gn的作者

如果您正在编辑 //build目录之外 的构建文件( 即,不直接处理工具链,编译器配置等),通常只需要担心一些事情:

该`current_toolchain`,`current_cpu`,和`current_os`变量反映了, 设置在 **目前** 给定的规则中有效.如`is_linux`,`is_win`等变量会更新以反映当前设置,并改为`cflags`，`ldflags`等等
仅适用于当前的工具链和当前正在构建的东西.

你也可以参考`target_cpu`和`target_os`变量.如果您需要在依赖`target_arch`需求的主机上执行不同的操作,这将非常有用; 所有工具链中的值都是不变的.你可以给`host_cpu`和`host_os`变量做类似的事情,但应该总是不需要的.

对于默认工具链,`target_cpu`和`current_cpu`是相同的.对于辅助工具链,`current_cpu`基于工具链定义和`target_cpu`设置会保持不变.在编写规则时, **大多数时候应该使用`current_cpu`而不是`target_cpu`**.

默认情况下,一条规则`rule`中具有依赖项列表的`deps`变量使用相同的(当前活动的)工具链.您可以指定不同的工具链,通过使用`foo(bar)`标签符号来描述,详细说明[参考文档的标签部分](reference.md#Toolchains).

这是一个何时使用`target_cpu`vs`current_cpu`的示例:

``` py
declare_args() {
  # 仅适用于定位target_cpu的工具链.
  sysroot = ""
}

config("my_config") {
  # 使用current_cpu，因为编译参数是工具链依赖的.
  if (current_cpu == "arm") {
    defines = [ "CPU_IS_32_BIT" ]
  } else {
    defines = [ "CPU_IS_64_BIT" ]
  }
  # 将current_cpu与target_cpu进行比较，
  # 看看current_toolchain是否与target_toolchain具有相同的体系结构.
  if (sysroot != "" && current_cpu == target_cpu) {
    cflags = [
      "-isysroot",
      sysroot,
    ]
  }
}
```

## 作为//build/config 或 //build/toolchain 作者

该`default_toolchain`声明在`//build/config/BUILDCONFIG.gn`文件.通常,`default_toolchain`应该是`target_os`和`target_cpu`的工具链.该`current_toolchain`反映了当前对规则有效的工具链.

一定要了解`host_cpu`,`target_cpu`,`current_cpu`,和`toolchain_cpu`(和os等价物) 它们之间的区别. 前两个设置[如上所述](#作为GN用户). 

你有责任确保`current_cpu`在工具链中正确设置;如果您使用的是存储的模板`gcc_toolchain`和`msvc_toolchain`,这意味着你有责任确保这一点`toolchain_cpu`和`toolchain_os`在模板调用中适当设置.
