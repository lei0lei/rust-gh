---
layout: default
title: rust-intro
permalink: /rust/语言圣经
parent: rust-course
has_toc: true
---
<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

# 安装

## linux
打开终端并输入下面命令：

```sh
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

这个命令将下载一个脚本并开始安装 rustup 工具，此工具将安装 Rust 的最新稳定版本。

Rust 对运行环境的依赖和 Go 语言很像，几乎所有环境都可以无需安装任何依赖直接运行。但是，Rust 会依赖 libc 和链接器 linker。所以如果遇到了提示链接器无法执行的错误，你需要再手动安装一个 C 语言编译器.

Linux 用户一般应按照相应发行版的文档来安装 GCC 或 Clang。

例如，如果你使用 Ubuntu，则可安装 build-essential。

## windows
Windows 上安装 Rust 需要有 C++ 环境，以下为安装的两种方式：

1. x86_64-pc-windows-msvc（官方推荐）

先安装 Microsoft C++ Build Tools，勾选安装 C++ 环境即可。安装时可自行修改缓存路径与安装路径，避免占用过多 C 盘空间。安装完成后，Rust 所需的 msvc 命令行程序需要手动添加到环境变量中，否则安装 Rust 时 `rustup-init` 会提示未安装 Microsoft C++ Build Tools，其位于：%Visual Studio 安装位置%\VC\Tools\MSVC\%version%\bin\Hostx64\x64（请自行替换其中的 %Visual Studio 安装位置%、%version% 字段）下。

如果你不想这么做，可以选择安装 Microsoft C++ Build Tools 新增的“定制”终端 Developer Command Prompt for %Visual Studio version% 或 Developer PowerShell for %Visual Studio version%，在其中运行 rustup-init.exe。

要卸载 Rust 和 rustup，在终端执行以下命令即可卸载：

```sh
$ rustup self uninstall
```

# cargo
Rust 语言的包管理工具是 cargo

```sh
$ cargo new world_hello
$ cd world_hello
```
上面的命令使用 cargo new 创建一个项目，项目名是 world_hello （向读者势力低头的项目名称，泪奔），该项目的结构和配置文件都是由 cargo 生成，意味着我们的项目被 cargo 所管理。

## 运行项目
有两种方式可以运行项目：

1. cargo run

2. 手动编译和运行项目

首先来看看第一种方式，一码胜似千言，在之前创建的 world_hello 目录下运行：

```
$ cargo run
   Compiling world_hello v0.1.0 (/Users/sunfei/development/rust/world_hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/world_hello`
Hello, world!
```

上述代码，cargo run 首先对项目进行编译，然后再运行，因此它实际上等同于运行了两个指令，下面我们手动试一下编译和运行项目：

- 编译
```
$ cargo build
    Finished dev [unoptimized + debuginfo] target(s) in 0.00s
```
- 运行
```
$ ./target/debug/world_hello
Hello, world!
```
在调用的时候，路径 ./target/debug/world_hello 中有一个明晃晃的 debug 字段，没错我们运行的是 debug 模式，在这种模式下，代码的编译速度会非常快，可是福兮祸所伏，运行速度就慢了. 原因是，在 debug 模式下，Rust 编译器不会做任何的优化，只为了尽快的编译完成，让你的开发流程更加顺畅。

## cargo check
当项目大了后，cargo run 和 cargo build 不可避免的会变慢，那么有没有更快的方式来验证代码的正确性呢？大杀器来了，接着！

cargo check 是我们在代码开发过程中最常用的命令，它的作用很简单：快速的检查一下代码能否编译通过。因此该命令速度会非常快，能节省大量的编译时间。

```
$ cargo check
    Checking world_hello v0.1.0 (/Users/sunfei/development/rust/world_hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.06s
```

## Cargo.toml 和 Cargo.lock
Cargo.toml 和 Cargo.lock 是 cargo 的核心文件，它的所有活动均基于此二者。

Cargo.toml 是 cargo 特有的项目数据描述文件。它存储了项目的所有元配置信息，如果 Rust 开发者希望 Rust 项目能够按照期望的方式进行构建、测试和运行，那么，必须按照合理的方式构建 Cargo.toml。

Cargo.lock 文件是 cargo 工具根据同一项目的 toml 文件生成的项目依赖详细清单，因此我们一般不用修改它，只需要对着 Cargo.toml 文件撸就行了。

什么情况下该把 Cargo.lock 上传到 git 仓库里？很简单，当你的项目是一个可运行的程序时，就上传 Cargo.lock，如果是一个依赖库项目，那么请把它添加到 .gitignore 中。

### package 配置段落
package 中记录了项目的描述信息，典型的如下：
```
[package]
name = "world_hello"
version = "0.1.0"
edition = "2021"
```
name 字段定义了项目名称，version 字段定义当前版本，新项目默认是 0.1.0，edition 字段定义了我们使用的 Rust 大版本。因为本书很新（不仅仅是现在新，未来也将及时修订，跟得上 Rust 的小步伐），所以使用的是 Rust edition 2021 大版本，详情见 [Rust 版本详解](https://course.rs/appendix/rust-version.html)

### 定义项目依赖
使用 cargo 工具的最大优势就在于，能够对该项目的各种依赖项进行方便、统一和灵活的管理。

在 Cargo.toml 中，主要通过各种依赖段落来描述该项目的各种依赖项：

- 基于 Rust 官方仓库 crates.io，通过版本说明来描述
- 基于项目源代码的 git 仓库地址，通过 URL 来描述
- 基于本地项目的绝对路径或者相对路径，通过类 Unix 模式的路径来描述
这三种形式具体写法如下：

```
[dependencies]
rand = "0.3"
hammer = { version = "0.5.0"}
color = { git = "https://github.com/bjz/color-rs" }
geometry = { path = "crates/geometry" }
```