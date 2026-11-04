---
image: "cover.svg"
social_image: "cover.jpg"
title: 中文路径下，Gradle 单测为什么会全挂
slug: gradle-path
date: 2026-10-29
description: release 编译完全正常，单测却 403 个全红，问题不在代码，在路径。
categories:
  - 技术
tags:
  - Android
  - Gradle
  - 踩坑
series:
  - liyan
---

我的工程放在一个中文路径下。

`assembleRelease` 跑得好好的，能出包、能装机。但一跑单元测试，**403 个用例全部失败**，清一色的 `ClassNotFoundException`。

那一刻我以为自己把代码写坏了。

---

## 排查顺序（都试过，都没用）

1. `clean` + 删 `.gradle` 缓存 —— 没用
2. 怀疑依赖冲突，重建依赖树 —— 没问题
3. 加 `jvmArgs '-Dfile.encoding=UTF-8'` —— 没用，一点变化都没有
4. 升级 AGP / Kotlin —— 没用

直到我注意到一个细节：**release 编译正常，只有测试挂。** 而且不是某个用例挂，是"连类都找不到"——说明压根不是代码问题，是测试进程启动时 classpath 就错了。

原因出在这里：Gradle 跑单测时会 **fork 一个新的 JVM 进程**，把一长串 classpath 当命令行参数传过去。路径里带中文时，这个参数在某些环节被按错误编码拆解，于是所有类都定位不到。

编译不 fork 进程，所以没事；测试 fork，所以全挂。

---

## 解法：别跟它较劲，绕过去

```bat
subst R: "c:\Users\someone\Desktop\理砚开发文件"
cd /d R:\理砚手机端
gradlew.bat testDebugUnitTest
subst R: /d
```

把一个 ASCII 盘符映射到中文目录，在 `R:\` 下跑，跑完删掉映射。全套写进一个 `.bat`，双击就行。

不优雅，但它彻底绕开了编码问题——不依赖任何 `-D` 参数，也不依赖 Gradle 版本。

---

## 两个会让你误判的细节

**一、`FROM-CACHE` 不等于跑过。**

Gradle 控制台出现 `testDebugUnitTest FROM-CACHE`，意思是这次直接复用了缓存结果，**根本没有执行测试**。如果你把一个坏掉的缓存当成"通过"，后面会很难查。

**二、看报告，不要只看控制台。**

控制台只给你摘要，具体哪个类没找到、哪个断言失败，要读 `app/build/test-results/` 里的 XML 报告。我当时就是盯着控制台猜了半天。

---

## 结论

**中文路径 + fork 的测试 worker，是工具链层面的系统性坑，不值得逐个参数去调。** 用盘符映射绕过去，把这套写进脚本，然后忘掉它。

如果你的项目路径里也有中文、空格或者特殊字符，而某个环节"莫名其妙地全红"，先怀疑路径，再怀疑代码。
