# 基于 Starklings 的交互式 Cairo 教程

在由 [WTF Academy](https://x.com/0xAA_Science/status/1721113230293234068) 举办的 Starknet中文训练营（Starknet Basecamp Chinese）中，我将为大家讲解 Cairo 基础教程。在开始之前，大家可以先对 Cairo 语言做一个大致的了解，并提前安装好 [Scarb](https://docs.swmansion.com/scarb/docs.html) 和 [Starklings](https://github.com/shramee/starklings-cairo1) 。我们将基于 Starklings 展开，通过交互式的学习来掌握 Cairo 语言的使用。


## 一、Cairo 语言简介


如 [Cairo 文档](https://book.cairo-lang.org/) 中所言，Cairo 是第一个图灵完备的，可用于创建可证明的通用计算程序的语言。它是一种类似 Rust 的高级语言。与 Rust 一样，它旨在让开发人员轻松编写高效且安全的代码。（*作者注：这里所讲的 Cairo 是指 Cairo 1.0 及之后的版本，其与初代的 Cairo 0 相比，语法上存在很大差异。* ）

Cairo 1.0 还引入了 Sierra，这是一种新的中间表示形式，可确保每次 Cairo 运行都能得到验证。这使得 Cairo 1.0 特别适合在 Starknet 这样的无需许可的网络中使用，它可以提供强大的 DoS 保护和审查抵抗能力。您可以在此处阅读有关 Sierra 的更多信息。

简单来说，Cairo 就是一种编写 **Starknet 智能合约** 的高级语言，如同 Solidity 之于 Ethereum 智能合约。从实践中来看，如果你有 Rust 基础的话，将会比较容易上手 Cairo。不过如果你之前没接触过 Rust 也没关系，本文将尽量用最通俗易懂的语言来讲解 Cairo 。

## 二、开发环境（Scarb）配置
编译型的语言需要编译器。正如 C 语言与其编译器 gcc , Rust 语言与其编译器 rustc 一样, Cairo 也有属于它的[编译工具](https://github.com/starkware-libs/cairo/releases), 我们可以直接下载这些二进制程序，使用命令 ```cairo-run --single-file HellWorld.cairo```来编译和运行源文件 HellWorld.cairo 。
 
 但以上只是 Cairo 的编译及运行的最小环境，为了更方便地进行项目开发，本文更推荐使用 [Scarb](https://docs.swmansion.com/scarb/docs.html) 作为代码构建及项目管理的工具：

 其文档中所说，Scarb 是一款 Cairo 包管理器。可以方便地下载 Cairo 包的依赖项，编译 Cairo 程序 或 StarkNet 合约，并作为其他工具（例如 Starknet Foundry 或 IDE）处理代码的入口。
*作者注：Scarb和 Cargo (用于 Rust 的构建工具和包管理器) 很相似。*
其 [安装方式](https://docs.swmansion.com/scarb/download.html) 如下 ：
(通过安装脚本进行安装，此方法仅适用于 macOS 和 Linux。如果你用的是 Windows10及以上系统，建议开启 WSL2 以获得 Linux 运行环境)
```shell
curl --proto '=https' --tlsv1.2 -sSf https://docs.swmansion.com/scarb/install.sh | sh
```
安装完毕后，通过命令
```shell
scarb new hello_world
```
就可以创建出一个名为 ```hello_world``` 的 Cairo 项目了。

注：以上示例创建了一个 Cairo 本地程序项目。如果项目要作为 Starknet 合约进行编译的话，需要在`Scarb.toml`中添加如下依赖:
```
[dependencies]
starknet = ">=2.1.0"

[[target.starknet-contract]]
```
想要了解更多的Scarb用法，可以查阅[官网文档](https://docs.swmansion.com/scarb/docs.html)。

## 三、初识 Cairo 程序和 Starknet 合约

先看一个最简单的 Cairo 程序：
```Rust
use debug::PrintTrait;

fn main() {
    'Hello, world!'.print();
}
```

再看一个简单的 Starknet 合约：
```Rust
#[starknet::interface]
trait ISimpleStorage<TContractState> {
    fn set(ref self: TContractState, x: u128);
    fn get(self: @TContractState) -> u128;
}

#[starknet::contract]
mod SimpleStorage {
    use starknet::get_caller_address;
    use starknet::ContractAddress;

    #[storage]
    struct Storage {
        stored_data: u128
    }

    #[external(v0)]
    impl SimpleStorage of super::ISimpleStorage<ContractState> {
        fn set(ref self: ContractState, x: u128) {
            self.stored_data.write(x);
        }
        fn get(self: @ContractState) -> u128 {
            self.stored_data.read()
        }
    }
}
```
可以发现， 常规的 Cairo 程序需要要有一个 main() 函数；而Starknet 合约则不需要 main() 函数，但却多了 #[storage] 一类的宏定义。（*作者注：实质上, Starknet 合约就是一个 Cairo module*）。

我们就先以 Cairo 程序作为学习切入点，了解 Cairo 的基本语法和数据类型，直到最终搭建起一个 Starknet 合约。

+ 小练习
可以用上面的 Cairo 程序覆盖掉 ```hello_world/src/lib.cairo```里的内容，然后在 hello_world/ 目录下执行
```shell
scarb cairo-run
```
看看这个 Cairo 程序的运行结果。

## 四、练习环境（Starklings）的配置
通过以上的学习，我们对 Cairo 语言有了一定的了解，为了更好地巩固和应用我们所学的内容，我们介绍一款可以交互练习 Cairo 和 Starknet 的工具： [Starklings](https://github.com/shramee/starklings-cairo1)   并将其安装到本地，用来巩固我们所学的知识。

由于这款工具是用 Rust 编写的而且需要在本地编译运行，所以我们首先要安装 Rust 及其包管理器 Cargo :
```shell
curl https://sh.rustup.rs -sSf | sh -s
```
然后 Git Clone 这个项目,并进入 starklings-cairo1 目录下（ 请确保你安装了 Git ）:
```shell
git clone https://github.com/shramee/starklings-cairo1.git && cd starklings-cairo1
```
在 starklings-cairo1 目录下执行以下命令，以编译并运行 Starklings 。首次运行需要下载很多依赖，所以用时会比较长：
```shell
cargo run -r --bin starklings
```
编译完成后，就可以执行以下命令开始练习了：
```shell
cargo run -r --bin starklings watch
```
更多帮助说明可以查看 [Starklings](https://github.com/shramee/starklings-cairo1) 主页。



**Reference:**
1. [Cairo](https://github.com/starkware-libs/cairo)
2. [Cairo Book](https://book.cairo-lang.org)
3. [WTF-Cairo](https://github.com/WTFAcademy/WTF-Cairo)
4. [Scarb Docs](https://docs.swmansion.com/scarb/docs.html)
5. [Starkling Cairo1](https://github.com/shramee/starklings-cairo1)