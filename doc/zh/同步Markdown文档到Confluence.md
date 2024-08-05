<!-- Space: documents -->
<!-- Parent: actions -->
<!-- Title: 同步Markdown文档到Confluence.md -->

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

## 前置说明
- 由于本文档也配置了 `mark metadata` 信息，文档中的某些标记会被强制替换，所以这边先做如下说明
    1. 脚本中的所有 `:TOC:` 中的 `TOC` 在使用时需要小写
    2. 脚本中的所有 `$$Macro` 需要移除 `$$`

## 工具列表
- [Github](https://github.com)
- [Github Action][github actions]
- [Mark][mark]
- [Confluence API Token][confluence api token]

## Markdown 文件添加 `metadata`
```xml
<!-- Space: documents -->
<!-- Parent: actions -->
<!-- Title: markdown to confluence by action -->

<!-- $$Macro: \!\[.*\]\((.+)\)\<\!\-\- width=(.*) \-\-\>
     Template: ac:image
     Url: ${1}
     Width: ${2} -->
<!-- $$Macro: :TOC:
     Template: ac:toc
     Printable: 'false'
     MinLevel: 2 
     MaxLevel: 4 -->
<!-- Include: doc/other/disclaimer.md -->

:TOC:
```
### `metadata` 说明
- 完整说明可参考：[Mark — a tool for syncing your markdown documentation with Atlassian Confluence pages.][mark]

#### `metadata` 基础信息说明
```xml
<!-- Space: documents -->
<!-- Parent: actions -->
<!-- Parent: guide -->
<!-- Title: markdown to confluence by action -->
```
- `Space`: 空间标识，获取 `空间管理 > 概览 > 空间细节 > 标识` 字段填入
- `Parent`: 父页面名称，多个按顺序查找，以上例子将会把文档放入 `actions > guide` 下
- `Title`: 生成页面的文件名称
#### `metadata` 自动配置图片大小（URL模式）
```xml
<!-- $$Macro: \!\[.*\]\((.+)\)\<\!\-\- width=(.*) \-\-\>
     Template: ac:image
     Url: ${1}
     Width: ${2} -->
```
- 原理是在图片标记后面添加宽度标记，例子如下
    - `![image1](http://test.com/a.png)<!-- width=750 -->`
- **注意**：在 action 中会自动在匹配的图片后面添加 width 标识，默认宽度为 750
    - 例如，原内容 `![image1](http://test.com/a.png)` 会被 `actions` 替换为 `![image1](http://test.com/a.png)<!-- width=750 -->`

#### ~~`metadata` 自动配置图片大小（Path模式） `内联模式使用该方法会导致附件无法自动上传，故废弃，不进行图片大小配置`~~
- **开启该功能后附件无法自动上传，需要手动上传，如果需要的话可以开启**
- 建议使用图床上传图片后，使用 url 模式
```xml
<!-- $$Macro: \!\[.*\]\((.+)\)\<\!\-\- width-attach=(.*) \-\-\>
     Template: ac:image
     Attachment: ${1}
     Width: ${2} -->
```
- 原理是在图片标记后面添加宽度标记，例子如下
    - `![image1](./test.com/a.png)<!-- width=750 -->`
- **注意**：在 action 中会自动在匹配的图片后面添加 width 标识，默认宽度为 750
    - 例如，原内容 `![image1](./test.com/a.png)` 会被 `actions` 替换为 `![image1](./test.com/a.png)<!-- width-attach=750 -->`

#### `metadata` 自动配置 `toc`
```xml
<!-- $$Macro: :TOC:
     Template: ac:toc
     Printable: 'false'
     MinLevel: 2 
     MaxLevel: 4 -->
:TOC:
```

## 仓库配置
### 添加 `workflows`
```yaml
name: Publish to Confluence
on:
  push:
    branches: [ master ]
    # paths:
    #   - '**'
    #   - '!.gitignore'
    #   - '!README.md'
  # workflow_dispatch: # 允许手动触发工作流
  # schedule:
  #   # 每天晚六点十分执行一次
  #   - cron:  '10 10 * * *'

env:
  SEPARATOR: "," # 定义多文件分割符
  START_PREFIX: "<!--" # 定义需要同步的文件

jobs:
  confluence:
    runs-on: ubuntu-latest

    steps:
      # 检出文件
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # 获取变更的文件
      - name: Get changed files
        id: changed-markdown-files
        uses: tj-actions/changed-files@v44.5.5
        with:
          files: |
            **.md
            **.png
          files_ignore: |
            **.tpl.md
          quotepath: false
          separator: ${{env.SEPARATOR}}
          old_new_files_separator: ";"

      # 打印所有变更的文件
      - name: Print all changed files
        if: steps.changed-markdown-files.outputs.any_changed == 'true'
        env:
          ALL_CHANGED_FILES: ${{ steps.changed-markdown-files.outputs.all_changed_files }}
        run: |
          IFS=${SEPARATOR}
          for file in ${ALL_CHANGED_FILES}; do
            echo "$file was changed, realfile $(echo $file | tr -d '\\')"
          done

      # 构建添加 mark 指令
      - name: Setup mark
        if: steps.changed-markdown-files.outputs.any_changed == 'true'
        uses: fabasoad/setup-mark-action@v1.0.1

      # 编译以及发布到
      - name: Mark and to confluence
        if: steps.changed-markdown-files.outputs.any_changed == 'true'
        env:
          ALL_CHANGED_FILES: ${{ steps.changed-markdown-files.outputs.all_changed_files }}
          MARK_USER: ${{ secrets.ATLASSIAN_USERNAME }}
          MARK_PASS: ${{ secrets.ATLASSIAN_API_TOKEN }}
          MARK_URL: ${{ secrets.ATLASSIAN_BASE_URL }}
        run: |
          IFS=${SEPARATOR}
          for file in ${ALL_CHANGED_FILES}; do
            if [[ $(head -n 1 $(echo $file | tr -d '\\')) =~ ^$START_PREFIX ]]; then
              # 打印文件第一行信息
              head -n 1 $(echo $file | tr -d '\\')
              # 配置图片大小为 750
              # sed -i -E 's/(.*\!\[.*\]\(.*\))([ ]*<\!\-\-[ ]*width=[0-9]*[ ]*\-\->)*/\1<!-- width=750 -->/g' $(echo $file | tr -d '\\')
              # path 替换
              # sed -i -E 's/(.*\!\[.*\]\([ \t]*([a-zA-Z]\:)?(\.*(\/|\\))?([^:\r\n\t\!]*\.[a-zA-Z]+)[ \t]*\))([ ]*<\!\-\-[ \t]*width=[0-9]*[ \t]*\-\->)*/\1<!-- width-attach=750 -->/g' $(echo $file | tr -d '\\')
              # url 替换
              sed -i -E 's/(.*\!\[.*\]\([ \t]*(http(s)?:\/\/)([^:\r\n\t\!]*(\.[a-zA-Z]+)?)[ \t]*\))([ ]*<\!\-\-[ \t]*width=[0-9]*[ \t]*\-\->)*/\1<!-- width=750 -->/g' $(echo $file | tr -d '\\')
              # 构建及发布
              mark --ci --debug --trace --include-path $(pwd) -p $MARK_PASS -b $MARK_URL -f $(echo $file | tr -d '\\') || exit 1;
            else
              echo "$(echo $file | tr -d '\\') not config metadata info, ignore file"
            fi
          done
```

### 添加 `Actions Secrets`
```yml
ATLASSIAN_USERNAME: username
ATLASSIAN_API_TOKEN: api token
ATLASSIAN_BASE_URL: confluence base url
```

### 测试是否成功


## 参考文献
- [changed files][changed files]
- [mark][mark]
- [confluence doc][confluence doc]
- [github actions][github actions]

[changed files]: https://github.com/tj-actions/changed-files
[mark]: https://github.com/kovetskiy/mark
[confluence doc]: https://confluence.atlassian.com/doc/code-block-macro-139390.html
[github actions]: https://docs.github.com/zh/actions/about-github-actions/understanding-github-actions
[confluence api token]: https://support.atlassian.com/atlassian-account/docs/manage-api-tokens-for-your-atlassian-account/