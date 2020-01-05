---
layout: post
title: 我使用Homebrew记录
categories: [tools, mac, packages]
tags: [tools, mac, packages]
fullview: true
markdown: kramdown

---

homebrew 官方地址：[https://brew.sh/index_zh-cn](https://brew.sh/index_zh-cn).


### 安装homebrew:

```sh
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

使用 Homebrew 安装 Apple（或您的 Linux 系统）没有预装但 你需要的东西。

```sh
$ brew install wget
```

Homebrew 会将软件包安装到独立目录，并将其文件软链接至 /usr/local 。

```sh
$ cd /usr/local
$ find Cellar
Cellar/wget/1.16.1
Cellar/wget/1.16.1/bin/wget
Cellar/wget/1.16.1/share/man/man1/wget.1

$ ls -l bin
bin/wget -> ../Cellar/wget/1.16.1/bin/wget
```

完全基于 Git 和 Ruby，所以自由修改的同时你仍可以轻松撤销你的变更或与上游更新合并。

```sh
$ brew edit wget # 使用 $EDITOR 编辑!

```

Homebrew 的配方都是简单的 Ruby 脚本：


```ruby
class Wget < Formula
  homepage "https://www.gnu.org/software/wget/"
  url "https://ftp.gnu.org/gnu/wget/wget-1.15.tar.gz"
  sha256 "52126be8cf1bddd7536886e74c053ad7d0ed2aa89b4b630f76785bac21695fcd"

  def install
    system "./configure", "--prefix=#{prefix}"
    system "make", "install"
  end
end
```

### 我的安装列表

```
$ brew list -1
ack
gdbm
gettext
git
icu4c
libevent
libidn2
libunistring
libyaml
ncurses
node
openssl@1.1
pcre2
pkg-config
python
readline
ruby
sqlite
thrift@0.9
tmux
tmuxinator
tree
wget
xz
zplug

```

### 遇到过的问题:

* thrift 安装低版本.

  修改Homebrew的Ruby 脚本。 `git clone https://github.com/Homebrew/homebrew-core` 下载到本地，找到thrift 低版本的formula, 然后, `brew install <formula>`。

*  磁盘空间不足:

    每天运行一次
    ```sh
    #!/bin/bash
    brew update && brew upgrade && brew cleanup
    ```
