## nft-floor-notifier

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
本项目是一个生成随机项目的项目。
主要目的是模仿一个个人项目的实现流程，每完成一部分，就提交到github。
任何项目类型都可以，web2的和web3的各占50%概率，
工具类，交易类，ai类，bot类，脚本类，科学类，任何学科都可以，教程类，学习类。但是都是个人solo项目的
github账号和ssh我已经设置好了，只需要push就可以了。
只是写代码，不需要实际运行，不用build，不用测试。可以写测试代码等各种运维代码，但是无需真的去运行

## 步骤
1. 首先，你得先说你要做什么项目，取个名字。然后自己往下继续无需我同意
2. 模仿实际项目，完成一部分就进行git提交
3. 必须用以下方式修改提交时间，下面是实例，不是真的时间
export GIT_AUTHOR_DATE="2025-02-01T10:30:00"
export GIT_COMMITTER_DATE="2025-02-01T10:30:00"
4. 每次提交，都修改提交时间到上一次提交时间的随机时间之后。时间必须很随机，不能是整点整秒，太假了。首次提交时间，也需要是项目开始时间之后的随机时间
7. 提交时间需要考虑到是个人业余项目，都是业余时间做的，所以间隔会很久，花时间会更长。每个项目至少提交6次以上
8. 必须不能让别人看出来是ai写的，是真人开发的。commit需要像真人写的
9. commit前，需要设置当前用户名和邮箱
git config user.name "whitakerluck"
git config user.email "tkjxohl6148@hotmail.com"
10. 你自己审核并push，符合所有要求后，就直接push到远端。然后继续写

项目开始时间2025-04-11T12:33:31，按照AGENTS.md里的需求，完成项目

## 注意事项
1. 尽量提高commit次数
2. 项目每次只做一点
3. 必须不能让别人看出来是ai写的，是真人开发的。commit需要像真人写的
4. .gitignore里也需要添加内容，把里面已有的内容，随机插入到其他行里
5. commit不能有ai的样子

## 提交完成后的检查
1. 要从github上拉取所有的提交数据，检查提交时间是不是修改为过去的时间，如果不是，就必须解决
2. 除了拉取检查提交时间，还要审核commit的内容，是不是像真人写的，如果看出来是ai写的，必须回退修改

---
> Source: [whitakerluck/NFT-Floor-Notifier](https://github.com/whitakerluck/NFT-Floor-Notifier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
