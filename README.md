<!-- Space: documents -->
<!-- Parent: actions -->
<!-- Title: README.md -->

<!-- Macro: \!\[.*\]\((.+)\)\<\!\-\- width=(.*) \-\-\>
     Template: ac:image
     Url: ${1}
     Width: ${2} -->
<!-- Macro: :toc:
     Template: ac:toc
     Printable: 'false'
     MinLevel: 2 
     MaxLevel: 4 -->
<!-- Include: doc/other/disclaimer.md -->

:toc:

## 自用 Github Actions
- 自己编写的一些有用的 github action 工具

## [英文/English](doc/en/README_EN.md)

## 功能特性列表
- [X] 自动同步 `Markdown` 文件到 `Confluence`
  - 将 `Markdown` 语法转化为 `Confluence` 标记语言
  - 将转化后的内容通过 `rest api` 发布到 `Confluence`
  - 不需要 `Confluence` 安装 `Markdown` 插件
  - [使用手册](doc/zh/同步Markdown文档到Confluence.md)