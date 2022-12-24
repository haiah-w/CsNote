Go

[Download and install - The Go Programming Language](https://go.dev/doc/install)

# Rust

Ubuntu20.04版本

GCC:[编译器、链接器和库简介 –learncpp.com](https://www.learncpp.com/cpp-tutorial/introduction-to-the-compiler-linker-and-libraries/)

```shell
sudo apt install make build-essential gcc curl
```

官网安装：[Rusts](https://www.rust-lang.org/zh-CN/tools/install)

```shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

速度慢，代理：[RsProxy](https://rsproxy.cn/)

错误：error: linking with `cc` failed: exit status: 1

更改rustup使用的toolchain

```shell
rustup install nightly-2019-06-01-x86_64-unknown-linux-gnu
rustup default nightly-2019-06-01-x86_64-unknown-linux-gnu
```

错误：supported edition values are `2015` or `2018`, but `2021` is unknown

```shell
rustup show
rustup update
```

# NodeJs

官网：[Download | Node.js (nodejs.org)](https://nodejs.org/en/download/)

下载二进制文件

```shellag-0-1ggsang4hag-1-1ggsang4h
tar -xvf node-v18.12.0-linux-x64.tar.xz
```

解压后，做软连接到/usr/bin/下

```shell
sudo ln -s home/haiah/dev/node-v18.12.0-linux-x64/node /usr/bin/node
sudo ln -s /home/haiah/dev/node-v18.12.0-linux-x64/lib/node_modules/npm/bin/npm-cli.js /usr/bin/npm
sudo ln -s /home/haiah/dev/node-v18.12.0-linux-x64/lib/node_modules/npm/bin/npx-cli.js /usr/bin/npx
```
