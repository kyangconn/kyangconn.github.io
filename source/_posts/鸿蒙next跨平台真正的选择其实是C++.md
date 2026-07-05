---
title: "鸿蒙next跨平台真正的选择其实是?"
date: 2026-06-30
tags: ["HarmonyOS", "C++", "Qt", "跨平台", "AI生成"]
category: ["技术实践", "跨平台"]
description: "从 Go+Web、Wails、Tauri 到 C++/Qt/QML，重新想了一遍鸿蒙 next 跨平台应用到底该怎么选。"
---

最近打算做一个本地优先的 LaTeX 编辑器，Windows、Linux 桌面和 HarmonyOS Next 都要跑。听起来不算离谱的需求，但真开始选技术栈才发现，跨平台的核心问题根本不是"有没有一个框架能跑所有平台"，而是——**你到底愿意写几套 UI、几套业务逻辑、几套胶水代码。**

结论先放前面：在鸿蒙 Next 这个约束下，最终靠谱的选择是 C++。

不是情怀，是被现实筛出来的。

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

但它们都有一个共同的问题：没有真正解决 HarmonyOS Next 这一端。

Flutter 不是鸿蒙的原生一等路线；Avalonia 会引入 .NET；Slint 轻归轻，但复杂编辑器需要的那套桌面生态——PDF、进程管理、文件系统——还不如 Qt 成熟。而这些方案都会让项目变成一套 desktop UI、一套 ArkUI、一套 C++ core、再加两套绑定层。UI 现代了，工程边界更碎了。

## 最后绕回 C++

于是绕回了一个看起来最土的选择。

不是说 C++ 写 UI 爽。Qt Widgets 写现代 UI 确实不爽——响应式布局、组件库、主题 token、class 调样式、热更新，这些在 Web 里是常识，到了 Qt Widgets 里全要重新面对。

但 C++ 在这个项目里的优势是硬性的：

- Windows / Linux 桌面天然适配。
- Qt 有成熟的文件、进程、PDF、国际化、打包能力。
- core 可以写成真正的平台无关 C++。
- 通过 C ABI 暴露给 HarmonyOS NAPI 消费。
- 鸿蒙 UI 走 ArkUI，不需要硬塞一个非原生 UI 框架。
- 桌面 UI 可以从 Qt Widgets 迁到 QML，让体验现代一点。

最终的结构反而比前面任何方案都清晰：

```
core/        C++ 业务逻辑
capi/        C ABI，给 HarmonyOS NAPI 和其他绑定用
apps/desktop Qt Quick/QML 或 Qt Widgets
apps/harmony ArkUI + ArkTS
```

这就是"1.8 套核心代码 + 2 套 UI"。不完美，但已经是现实里最不拧巴的解。

## 为什么不共享 UI

因为 HarmonyOS Next 的正确路线就是 ArkUI。你当然可以想办法塞 WebView，或者等某些框架适配，但那样就不是"真正适配鸿蒙"，而是在鸿蒙上跑一个外来 UI 技术栈。对于一个本地编辑器来说，不值当。

更现实的做法是共享核心，而不是共享界面。

业务逻辑、路径安全、项目模型、文件读写、编译流程、SyncTeX、diff、snapshot——这些才应该共享。UI 本来就应该贴近平台。Windows/Linux 桌面和鸿蒙的操作习惯不一样，强行统一 UI 只会两边都不舒服。

## Qt Widgets 的痛苦和 QML 的诱惑

现在项目里最大的痛点不在核心功能，而在 UI 对齐颗粒度。

Qt Widgets 写起来像是在用代码手摆控件。按钮、下拉框、数字输入、表单行、窗口尺寸、圆角、hover 状态，每个都能调，但每个都不像 Web 组件库那样有现成抽象。核心功能 AI 能很快写出来，UI 细节却会在原生控件的默认行为里反复漏水。

后续桌面端大概率会从 Widgets 转向 Qt Quick/QML。不是因为 QML 完美，而是它至少提供了声明式 UI、Qt Quick Controls、style 系统，以及更接近前端的开发心智。不过就算转 QML，底层选择仍然是 C++。

## AI 写代码的强度已经变了

这次让我感触很深的是，AI 写代码已经不是"帮你补个函数"的水平了。它可以连续做这些事：改 CI 触发规则、修 CMake 和 Qt 依赖、排查 clang-format 问题、写 Qt Widgets 页面、接设置持久化、跑 lint 和 ctest、讨论架构选型，甚至顺手帮你写这篇博客。

但强度变高之后，另一个问题也更明显：**AI 不知道你没写出来的常识。**

比如"这个项目要同时给 HarmonyOS 和 Windows/Linux 用"这个前提，如果不写进 todo、docs、架构契约里，AI 就会按眼前代码推断。它不会天然知道你脑子的产品边界。

所以现在写项目，文档和 todo 不是给人看的备忘录，而是给 AI 对齐颗粒度的协议。你不写，它就猜。你写得模糊，它就按自己的默认经验补。

## 总结

如果只做桌面，我可能会选 Tauri。如果只做 HarmonyOS，我会老老实实写 ArkTS。

但如果目标同时包含 Windows/Linux 桌面和 HarmonyOS Next，要做轻量本地应用、共享核心逻辑、又不想绑死在 WebView 或 Node 生态里——那绕一圈回来，答案真的就是 C++。

不是因为 C++ 优雅，也不是 Qt Widgets 好写。而是在真正跨平台这件事上，最重要的不是 UI 写得有多爽，而是核心边界能不能稳住。

UI 可以两套，核心最好一套。
