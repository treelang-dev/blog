---
title: Android 项目配置 GitHub Actions 自动构建
author: Treelang
pubDatetime: 2026-05-11T12:00:00Z
slug: android-github-actions-build
featured: false
draft: false
tags:
  - Android
  - GitHub Actions
  - CI/CD
  - Java
description: 记录 Android 项目使用 GitHub Actions 实现自动化构建的过程，包括 workflow 文件配置、构建命令、APK 输出、密钥管理和常见问题处理。
---

## 前言

在 Android 项目开发过程中，每次手动打开 Android Studio、点击构建、导出 APK 会比较麻烦。如果项目需要频繁修改和发布测试版本，就更适合使用 GitHub Actions 进行自动化构建。

GitHub Actions 可以在代码推送后自动执行构建流程，并生成 APK 文件。这样不仅方便测试，也能让项目构建过程更加规范。本篇文章记录的是 Android 项目配置 GitHub Actions 自动构建的基本过程。

## 项目背景

我的 Android 项目主要使用 Java 和 XML 进行原生开发。项目平时会在本地 Android Studio 中调试，但为了方便版本管理和自动打包，希望将构建流程放到 GitHub Actions 中。

自动化构建的目标包括：

- 每次 push 后自动构建
- 可以手动触发构建
- 输出 debug APK
- 后续可以扩展 release 签名构建
- 避免在仓库中暴露密钥

## GitHub Actions 目录结构

GitHub Actions 的配置文件需要放在项目根目录下的：

```text
.github/workflows
```

例如：

```text
.github/workflows/android-build.yml
```

如果项目中没有 `.github` 文件夹，需要手动创建。

## 基础 workflow 配置

可以新建文件：

```text
.github/workflows/android-build.yml
```

内容如下：

```yaml
name: Android Build

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  build:
    name: Build Android Project
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 17

      - name: Grant execute permission for Gradle
        run: chmod +x ./gradlew

      - name: Build debug APK
        run: ./gradlew assembleDebug

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: app-debug
          path: app/build/outputs/apk/debug/*.apk
```

这个配置会在代码推送到 `main` 分支时自动构建，也可以在 GitHub 页面手动触发。

## 配置说明

`actions/checkout@v4` 用于拉取项目代码。

`actions/setup-java@v4` 用于配置 Java 环境。Android 项目通常需要 JDK，根据 Gradle 和 Android Gradle Plugin 的版本不同，可能需要 JDK 11 或 JDK 17。

`chmod +x ./gradlew` 是为了确保 Gradle Wrapper 有执行权限。

`./gradlew assembleDebug` 用于构建 debug APK。

`actions/upload-artifact@v4` 用于将构建出来的 APK 上传到 GitHub Actions 的 Artifacts 中，方便下载测试。

## 检查 Gradle Wrapper

项目根目录应该包含：

```text
gradlew
gradlew.bat
gradle/wrapper/gradle-wrapper.properties
```

如果没有这些文件，GitHub Actions 中就无法直接执行：

```bash
./gradlew assembleDebug
```

这时需要先在本地项目中生成 Gradle Wrapper，或者确保项目完整上传到 GitHub。

## 构建输出位置

常见的 debug APK 输出路径是：

```text
app/build/outputs/apk/debug/
```

如果你的模块名不是 `app`，需要修改 workflow 中的路径。例如模块名是 `mean`，路径可能变成：

```text
mean/build/outputs/apk/debug/*.apk
```

因此如果上传 Artifacts 时提示找不到文件，需要先检查实际构建输出路径。

## 手动触发构建

由于 workflow 中加入了：

```yaml
workflow_dispatch:
```

所以可以在 GitHub 仓库页面中进入：

```text
Actions
→ Android Build
→ Run workflow
```

手动触发一次构建。

这在测试 workflow 是否正常时比较方便，不需要每次都修改代码再 push。

## 密钥管理

如果只是构建 debug APK，一般不需要配置签名密钥。如果要构建 release APK，就不能直接把 keystore 文件和密码写进仓库。

比较安全的做法是使用 GitHub Secrets 保存敏感信息，例如：

- KEYSTORE_BASE64
- KEYSTORE_PASSWORD
- KEY_ALIAS
- KEY_PASSWORD

在 Actions 中再通过脚本还原 keystore 文件。

需要注意的是，任何 token、密码、cookie、签名文件都不应该直接提交到公开仓库中。

## 常见问题

### 1. gradlew 没有执行权限

报错可能类似：

```text
Permission denied
```

解决方法是在 workflow 中加入：

```yaml
- name: Grant execute permission for Gradle
  run: chmod +x ./gradlew
```

### 2. JDK 版本不匹配

如果构建时报 Java 版本错误，需要检查 Android Gradle Plugin 和 Gradle 版本。可以尝试将 JDK 改成 11 或 17。

例如：

```yaml
java-version: 17
```

### 3. 找不到 APK 文件

如果 upload-artifact 报找不到文件，需要检查实际输出目录。可以临时加入一条命令：

```yaml
- name: List build outputs
  run: find . -name "*.apk"
```

这样可以看到 APK 实际生成在哪里。

### 4. 依赖下载失败

有时 GitHub Actions 下载 Gradle 依赖速度可能不稳定。可以稍后重新运行，或者检查项目的仓库源配置是否正常。

## 后续优化

基础构建完成后，可以继续增加以下功能：

- release APK 构建
- 自动生成版本号
- 构建完成后发送 Telegram 通知
- 自动创建 GitHub Release
- 多渠道打包
- 缓存 Gradle 依赖提升构建速度

例如可以增加 Gradle 缓存，让构建速度更快。

## 总结

通过 GitHub Actions，可以把 Android 项目的构建流程自动化。这样每次修改代码后，只需要 push 到 GitHub，就可以自动得到构建结果。

对于个人项目来说，自动化构建不仅能节省手动打包时间，也能让项目更加规范。后续如果需要发布测试版本或正式版本，可以在这个基础上继续扩展签名、Release 和通知功能。
