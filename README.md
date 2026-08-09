# Android ARM64 构建修复方案

解决在 ARM64 (aarch64) 架构上构建 Android 项目时遇到的 AAPT2 `Illegal instruction` 问题。

## 问题原因

Android SDK 的 AAPT2 工具仅提供 x86_64 架构的 Linux 二进制文件。当在 ARM64 环境中运行时，会因指令集不兼容而崩溃：

```
Illegal instruction (core dumped) [aapt2]
exit code 132
```

## 解决方案

### 方案 1：GitHub Actions (推荐 - 最简单)

在仓库中添加 `.github/workflows/build.yml`，使用 x86_64 runner：

```yaml
name: Android Build
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest  # x86_64 runner
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
      - run: ./gradlew assembleDebug
```

**优点**：无需本地配置，直接推送到 GitHub 即可自动构建

### 方案 2：安装 Box64

```bash
# 安装依赖
sudo apt-get update
sudo apt-get install -y wget git build-essential cmake

# 编译安装 Box64
git clone --depth 1 https://github.com/ptitSeb/box64.git
cd box64 && mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=RelWithDebInfo
make -j$(nproc)
sudo make install
sudo ldconfig

# 使用
BOX64_LOG=0 ./gradlew assembleDebug
```

### 方案 3：ARM64 原生构建工具

```bash
curl -fsSL https://raw.githubusercontent.com/Commit451/android-arm-build-tools/main/install.sh | bash
```

## 使用步骤

1. 将 `.github/workflows/build.yml` 添加到你的 Android 项目根目录
2. 推送代码到 GitHub
3. 在 Actions 标签页查看构建结果
4. 下载生成的 APK（在 Actions 页面的 artifacts 中）

## 资源链接

- [GitHub Actions 文档](https://docs.github.com/en/actions)
- [Box64 GitHub](https://github.com/ptitSeb/box64)
- [ARM64 构建工具](https://github.com/Commit451/android-arm-build-tools)
