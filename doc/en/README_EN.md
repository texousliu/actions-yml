<!-- Space: documents -->
<!-- Parent: actions -->
<!-- Title: README_EN.md -->

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

## Github Actions for personal use
- Some useful github action tools written by myself

## [中文/Chinese][README]

## Feature list
- [X] Automatically synchronize `Markdown` files to `Confluence`
  - Convert `Markdown` syntax to `Confluence` markup language
  - Publish the converted content to `Confluence` via `rest api`
  - No need to install `Markdown` plugin in `Confluence`
  - [User Manual][SyncMarkdownToConfluence]


[README]: ../../README.md
[SyncMarkdownToConfluence]: ./SyncMarkdownToConfluence.md
