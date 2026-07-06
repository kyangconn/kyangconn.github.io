---
title: "鸿蒙next跨平台真正的选择其实是?"
date: 2026-06-30
tags: ["HarmonyOS", "C++", "Qt", "跨平台", "AI生成"]
category: ["技术实践", "跨平台"]
description: "从 Go+Web、Wails、Tauri 到 C++/Qt/QML，重新想了一遍鸿蒙 next 跨平台应用到底该怎么选。"
---

最近打算做一个本地优先的 LaTeX 编辑器，Windows、Linux 桌面和 HarmonyOS Next 都要跑。听起来不算离谱的需求，但真开始选技术栈才发现，跨平台的核心问题根本不是"有没有一个框架能跑所有平台"，而是——**你到底愿意写几套 UI、几套业务逻辑、几套胶水代码。**

结论先放前面：在鸿蒙 Next 这个约束下，最终靠谱的底座还是 C++，但 UI 这件事现在没法像之前那样一口咬死“两套”了。

因为 Qt 官方 Wiki 里已经出现了 Qt for HarmonyOS，甚至有一页专门写 Qt6 for HarmonyOS 的构建流程。事情一下就从“鸿蒙端只能 ArkUI”变成了“Qt 可能也能往鸿蒙上打，只是你敢不敢现在押”。

{% grid %}

<!-- cell -->

{% link https://wiki.qt.io/Qt_for_HarmonyOS Qt for HarmonyOS desc:true %}

<!-- cell -->

{% link https://wiki.qt.io/Building_Qt6_for_HarmonyOS Building Qt6 for HarmonyOS desc:true %}

{% endgrid %}

{% folding color:yellow 旧版判断：1.8 套核心代码 + 2 套 UI %}

旧版本里我的判断更保守：桌面走 Qt，HarmonyOS 走 ArkUI，底层通过 C ABI/NAPI 共享 core。这个判断当时不算错，因为公开可见的 Qt/Harmony 路线确实不够明确。

但现在 Qt 官方已经把 HarmonyOS 相关 Wiki、构建说明、模块适配、平台限制这些东西挂出来了。它还不等于“生产可用、随便上”，但至少说明这条路不是纯脑补了。

{% endfolding %}

不是情怀，是被现实筛出来的。只是现实又更新了一版。

## 从 Go+Web 开始想

最开始的想法其实很自然：Go 写后端逻辑，Vue 写 UI。Web 生态成熟，组件库一抓一大把，调样式也舒服。但 LaTeX 编辑器不是后台管理系统——它要处理文件树、编辑器、PDF 预览、编译进程、项目状态、快捷键、拖拽、本地路径。Web 当然都能做，但越做越像"在浏览器里模拟一个桌面应用"。

于是就想到桌面壳。

## Wails 和 Tauri，都卡在同一个地方

Wails 的想法挺好，Go + WebView，比 Electron 轻。但实际用下来，生态和能力不如 Tauri，社区资料也没那么足。而且一旦 HarmonyOS 要另写一套，Wails 的优势就没什么意思了。

Tauri 比 Wails 成熟得多，Rust 生态、插件、社区都强。正常来说很可能是答案——如果目标只有 Windows / Linux / macOS 的话。

问题就在鸿蒙这里。HarmonyOS Next 的应用模型、ArkUI、ArkTS、NAPI 都不是普通桌面 WebView 能顺手覆盖的。最后很容易变成：

```
桌面：Tauri + Web
鸿蒙：ArkUI + ArkTS
核心：Rust / Go / C++ 之间再桥一层
```

复杂度没减少，反而拆成了更多语言、更多桥、更多构建系统。

## Flutter、Avalonia、Slint 也都不完美

Flutter 桌面能跑，Dart FFI 可以接 C/C++。Avalonia 的桌面 UI 很现代，XAML/MVVM 成熟。Slint 更轻，跟 C++/Rust 结合也很漂亮。

但它们都有一个共同的问题：HarmonyOS Next 这一端不够顺。

Flutter 不是鸿蒙的原生一等路线；Avalonia 会引入 .NET；Slint 轻归轻，但复杂编辑器需要的那套桌面生态——PDF、进程管理、文件系统——还不如 Qt 成熟。

之前我会直接说：这些方案最后都会变成一套 desktop UI、一套 ArkUI、一套 C++ core、再加两套绑定层。UI 现代了，工程边界更碎了。

现在这个说法得给 Qt 留个口子。Qt6 for HarmonyOS 如果能跑起来，至少理论上可以把桌面和 HarmonyOS 的 UI 也往一套 Qt/QML 上靠。问题是这玩意儿目前看起来更像“可以动手试”的工程路线，不像“闭眼上车”的稳定答案。

## 最后绕回 C++

于是绕回了一个看起来最土的选择。

不是说 C++ 写 UI 爽。Qt Widgets 写现代 UI 确实不爽——响应式布局、组件库、主题 token、class 调样式、热更新，这些在 Web 里是常识，到了 Qt Widgets 里全要重新面对。

但 C++ 在这个项目里的优势是硬性的：

- Windows / Linux 桌面天然适配。
- Qt 有成熟的文件、进程、PDF、国际化、打包能力。
- core 可以写成真正的平台无关 C++。
- 如果 Qt for HarmonyOS 走得通，HarmonyOS 端也许可以直接吃 Qt/QML。
- 如果 Qt for HarmonyOS 走不通，还可以退回 C ABI 暴露给 HarmonyOS NAPI，再写 ArkUI。
- 桌面 UI 可以从 Qt Widgets 迁到 Qt Quick/QML，让体验现代一点。

最终的结构反而比前面任何方案都清晰：

```
core/        C++ 业务逻辑
capi/        C ABI，保底给 HarmonyOS NAPI 和其他绑定用
apps/desktop Qt Quick/QML
apps/harmony 优先尝试 Qt/QML，必要时退回 ArkUI + ArkTS
```

这就从"1.8 套核心代码 + 2 套 UI"变成了"一套核心，UI 先押 Qt/QML，但别把退路拆了"。

## UI 到底能不能共享

以前我会说，HarmonyOS Next 的正确路线就是 ArkUI。你当然可以想办法塞 WebView，或者等某些框架适配，但那样就不是"真正适配鸿蒙"，而是在鸿蒙上跑一个外来 UI 技术栈。对于一个本地编辑器来说，不值当。

现在这句话要改得谨慎一点。

Qt for HarmonyOS 出现之后，UI 共享不再是完全没谱。尤其是如果桌面端本来就准备转 Qt Quick/QML，那 HarmonyOS 端也尝试 Qt/QML 就很自然。它至少比“桌面 Tauri、鸿蒙 ArkUI、核心再桥一层 Rust/Go/C++”听起来少点精神污染。

但是我还是不想把话说满。官方 Wiki 有构建说明、有 SDK 要求、有额外依赖包，也提到了平台限制和模块适配状态。这些信息的潜台词很明显：能玩，但别装作它已经跟 Windows/Linux 桌面一样成熟。

所以更稳的做法不是“共享 UI”或者“不共享 UI”二选一，而是：

1. core 必须独立，别让 UI 框架把业务逻辑绑死。
2. desktop 先往 Qt Quick/QML 走。
3. HarmonyOS 先试 Qt/QML，记录能跑到什么程度。
4. 如果遇到平台限制，再用 C ABI/NAPI 回退到 ArkUI。

业务逻辑、路径安全、项目模型、文件读写、编译流程、SyncTeX、diff、snapshot——这些才应该共享。UI 本来就应该贴近平台。Windows/Linux 桌面和鸿蒙的操作习惯不一样，强行统一 UI 只会两边都不舒服。

## Qt Widgets 的痛苦和 QML 的诱惑

现在项目里最大的痛点不在核心功能，而在 UI 对齐颗粒度。

Qt Widgets 写起来像是在用代码手摆控件。按钮、下拉框、数字输入、表单行、窗口尺寸、圆角、hover 状态，每个都能调，但每个都不像 Web 组件库那样有现成抽象。核心功能 AI 能很快写出来，UI 细节却会在原生控件的默认行为里反复漏水。

后续桌面端大概率会从 Widgets 转向 Qt Quick/QML。不是因为 QML 完美，而是它至少提供了声明式 UI、Qt Quick Controls、style 系统，以及更接近前端的开发心智。

现在 Qt6 for HarmonyOS 的 Wiki 冒出来之后，QML 的诱惑更大了。之前 QML 只是“让桌面端现代一点”，现在它可能顺手变成“试着把 HarmonyOS 也纳进来”的入口。

当然，这里有个很现实的问题：Wiki 能构建，不等于你的复杂应用能舒服地跑。一个 LaTeX 编辑器不是 Gallery 示例，它要文件系统、进程、PDF、输入法、窗口生命周期、路径权限、后台任务。每一项都可能在 HarmonyOS 上冒出一个“你以为你会了”的新坑。

## AI 写代码的强度已经变了

这次让我感触很深的是，AI 写代码已经不是"帮你补个函数"的水平了。它可以连续做这些事：改 CI 触发规则、修 CMake 和 Qt 依赖、排查 clang-format 问题、写 Qt Widgets 页面、接设置持久化、跑 lint 和 ctest、讨论架构选型，甚至顺手帮你写这篇博客。

但强度变高之后，另一个问题也更明显：**AI 不知道你没写出来的常识。**

比如"这个项目要同时给 HarmonyOS 和 Windows/Linux 用"这个前提，如果不写进 todo、docs、架构契约里，AI 就会按眼前代码推断。它不会天然知道你脑子的产品边界。

所以现在写项目，文档和 todo 不是给人看的备忘录，而是给 AI 对齐颗粒度的协议。你不写，它就猜。你写得模糊，它就按自己的默认经验补。

## 总结

如果只做桌面，我可能会选 Tauri。如果只做 HarmonyOS，我以前会老老实实写 ArkTS，现在会先看 Qt for HarmonyOS 能不能把我少折磨一点。

但如果目标同时包含 Windows/Linux 桌面和 HarmonyOS Next，要做轻量本地应用、共享核心逻辑、又不想绑死在 WebView 或 Node 生态里——那绕一圈回来，答案真的就是 C++。

不是因为 C++ 优雅，也不是 Qt Widgets 好写。而是在真正跨平台这件事上，最重要的不是 UI 写得有多爽，而是核心边界能不能稳住。

现在的答案更新成这样：

```
核心：C++，必须稳住。
桌面：Qt Quick/QML。
HarmonyOS：先试 Qt/QML，失败再退 ArkUI + NAPI。
```

UI 也许可以一套，但退路最好也算一套。
