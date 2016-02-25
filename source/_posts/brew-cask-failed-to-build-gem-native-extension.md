---
title: Homebrew cask 触发 Failed to build gem native extension 错误解决
date: 2016-02-24 19:12:19
tags:
		- brew cask
		- ruby
		- gem
		- 异常
---

### 问题

今天在使用 homebrew cask 安装软件时，报如下错误：

~~~bash
Error: ERROR: Failed to build gem native extension.

    /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/bin/ruby extconf.rb
checking for main() in -lpq... yes
checking for libpq-fe.h... no
Could not find PostgreSQL build environment (libraries & headers): Makefile not created

extconf failed, exit code 1

Gem files will remain installed in /Library/Ruby/Gems/2.0.0/gems/do_postgres-0.10.16 for inspection.
Results logged to /Library/Ruby/Gems/2.0.0/extensions/universal-darwin-15/2.0.0/do_postgres-0.10.16/gem_make.out
~~~
<!--more-->
### 排查
看到这个错误提示，应该是 PostgreSQL 相关的 gem 在安装时未正确安装，于是乎立马 `gem list --local` 看看，发现并没有安装 PostgreSQL 相关的 gem。经过一番排查，发现应该是在 Mac 上使用 homebrew 安装过 ruby，而出问题的 gem 是系统自带的 gem 安装的，当时安装失败而没有处理。Mac 系统升级到 EI Captain 之后使用 ruby 各种不舒服，于是用 homebrew 又安装了一个新版本的 ruby，就造成今天这个问题。

### 解决
用 homebrew 卸载 ruby 之后，再卸载 PostgreSQL 相关的 gem 之后，再用 homebrew 重新安装 ruby，最后 `brew cask` 成功。