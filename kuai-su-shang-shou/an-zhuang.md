---
description: LCUI 的安装方法以及版本更新相关说明。
---

# 安装

#### 语义化版本控制 <a id="&#x8BED;&#x4E49;&#x5316;&#x7248;&#x672C;&#x63A7;&#x5236;"></a>

LCUI 在其所有项目中公布的功能和行为都遵循[语义化版本控制](https://semver.org/lang/zh-CN/)。对于未公布的或内部暴露的行为，其变更会描述在[发布说明](https://github.com/lc-soft/LCUI/releases)中。

#### 更新日志 <a id="&#x66F4;&#x65B0;&#x65E5;&#x5FD7;"></a>

最新稳定版本：2.1.0

每个版本的更新日志见 [GitHub](https://github.com/lc-soft/LCUI/releases)。

### 直接用已编译好的成品

通常新版本的[发布说明](https://github.com/lc-soft/LCUI/releases)中都会附带已编译的二进制文件包，包括适用于 Ubuntu 系统的 deb 安装包和适用于 Windows 系统的 zip 包，你可以根据自己的需求下载，然后配置编译器的头文件和库文件的搜索路径以及链接器参数。

### LCPkg 包管理器

LCpkg 是一个用于管理 C/C++ 项目依赖的命令行工具，目前仅适合在 Windows 系统上使用，使用它你可以很方便的下载 LCUI 的二进制文件包。

```text
# 最新稳定版
lcpkg install github.com/lc-soft/LCUI
```

### 命令行工具 \(CLI\)

LCUI 提供了一个[官方的 CLI](https://github.com/lc-ui/lcui-cli)，为 LCUI 应用快速搭建繁杂的脚手架。更多详情可查阅 [LCUI CLI 的文档](https://github.com/lc-ui/lcui-cli)。

{% hint style="warning" %}
CLI 工具假定用户对 Node.js 和相关构建工具有一定程度的了解。如果你是新手，我们强烈建议先在不用构建工具的情况下通读指南，在熟悉 LCUI 本身之后再使用 CLI。
{% endhint %}



