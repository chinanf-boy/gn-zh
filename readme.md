# gn [![translate-svg]][translate-list] 

[translate-svg]: http://llever.com/translate.svg
[translate-list]: https://github.com/chinanf-boy/chinese-translate-list

「 GN是一个元构建系统,可以为[ninja](https://ninja-build.org)生成构建文件. 」

[中文](./readme.md) | [english](https://gn.googlesource.com/gn/)


---

## 校对 ✅

<!-- doc-templite START generated -->
<!-- docTempliteId = 'google' -->
<!-- repo = 'gn' -->
<!-- repo = 'gn' -->
<!-- commit = '77d64a3da6bc7d8b0aab83ff7459b3280e6a84f2' -->
<!-- time = '2018 9.16' -->
翻译的原文 | 与日期 | 最新更新 | 更多
---|---|---|---
[commit] | ⏰ 2018 9.16 | [googlesource] | [中文翻译][translate-list]

> 需要翻墙

[googlesource]: https://.googlesource.com/gn/+/master
[commit]: https://.googlesource.com/gn/+/77d64a3da6bc7d8b0aab83ff7459b3280e6a84f2
<!-- doc-templite END generated -->

- [x] [readme](./readme.md)
- [ ] [docs](./docs) 7/8
    - [./docs/faq.zh.md](./docs/faq.zh.md) 常见问题
    - [./docs/quick_start.zh.md](./docs/quick_start.zh.md) 快速入门
    - [./docs/cross_compiles.zh.md](./docs/cross_compiles.zh.md) 跨平台编译
    - [./docs/standalone.zh.md](./docs/standalone.zh.md) GN的简单独立构建
    - [./docs/update_binaries.zh.md](./docs/update_binaries.zh.md) 更新Chromium使用的GN二进制文件.
    - [./docs/language.zh.md](./docs/language.zh.md) gn语法设计
    - [./docs/style_guide.zh.md](./docs/style_guide.zh.md) gn 风格指南
    - [ ] [./docs/reference.zh.md](./docs/reference.zh.md) 😢 放弃 参考文件的翻译


### 贡献

欢迎 👏 勘误/校对/更新贡献 😊 [具体贡献请看](https://github.com/chinanf-boy/chinese-translate-list#贡献)

## 生活

[If help, **buy** me coffee —— 营养跟不上了，给我来瓶营养快线吧! 💰](https://github.com/chinanf-boy/live-need-money)

---

### 目录

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [GN](#gn)
  - [入门](#%E5%85%A5%E9%97%A8)
  - [发送补丁](#%E5%8F%91%E9%80%81%E8%A1%A5%E4%B8%81)
  - [社区](#%E7%A4%BE%E5%8C%BA)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


# GN

GN是一个元构建系统,可以为[ninja](https://ninja-build.org)生成构建文件.查看[文档/](./docs/quick_start.zh.md)了解更多.

## 入门

```bash
git clone https://gn.googlesource.com/gn
cd gn
python build/gen.py
ninja -C out
# To run tests:
out/gn_unittests
```

在Windows上,它预想三个`cl.exe`,`link.exe`,和`lib.exe`可以在`PATH`找到,因此您需要在Visual Studio命令提示符或类似命令运行运行.

在Linux和Mac上,默认编译器是`clang++`,最近的版本预想编译器在`PATH`可以找到.这可以覆盖通过设置`CC`,`CXX`,和`AR`.

## 发送补丁

GN使用[Gerrit](https://www.gerritcodereview.com/)用于代码审查.如何修补的简短版本是:

```
注册 在 https://gn-review.googlesource.com.

... 编辑代码后 ...
ninja -C out && out/gn_unittests
```

然后,上传更改以供审核:

```
git commit
git cl upload --gerrit
```

修改更改时,请使用:

```
git commit --amend
git cl upload --gerrit
```

这将添加新的更改到现有的代码审查,而不是创建一个新的.

我们要求所有贡献者[签署Google的贡献者许可协议](https://cla.developers.google.com/)(根据需要选择个人或公司,选择"任何其他Google项目").

## 社区

您可以提出问题,并跟随GN的开发,在Chromium上的[gn-dev@](https://groups.google.com/a/chromium.org/forum/#!forum/gn-dev)谷歌群.
