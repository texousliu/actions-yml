## Preface
- Since this document also configures `mark metadata` information, some tags in the document will be forcibly replaced, so here are the following instructions
  1. All `TOC` in `:TOC:` in the script need to be lowercase when used
  2. All `$$Macro` in the script need to remove `$$`

## Tool List
- [Github](https://github.com)
- [Github Action][github actions]
- [Mark][mark]
- [Confluence API Token][confluence api token]

## Add `metadata` to Markdown files
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
:TOC:
```
### `metadata` description
- For a complete description, see: [Mark — a tool for syncing your markdown documentation with Atlassian Confluence pages.][mark]

#### `metadata` Basic Information Description
```xml
<!-- Space: documents -->
<!-- Parent: actions -->
<!-- Parent: guide -->
<!-- Title: markdown to confluence by action -->
```
- `Space`: 空间标识，获取 `空间管理 > 概览 > 空间细节 > 标识` 字段填入
- `Parent`: 父页面名称，多个按顺序查找，以上例子将会把文档放入 `actions > guide` 下
- `Title`: 生成页面的文件名称

#### `metadata` automatically configures image size (URL mode)
```xml
<!-- $$Macro: \!\[.*\]\((.+)\)\<\!\-\- width=(.*) \-\-\>
     Template: ac:image
     Url: ${1}
     Width: ${2} -->
```
- The principle is to add a width tag after the image tag, as shown below
  - `![image1](http://test.com/a.png)<!-- width=750 -->`
- **Note**: In action, the width tag will be automatically added after the matching image, and the default width is 750
  - For example, the original content `![image1](http://test.com/a.png)` will be replaced by `![image1](http://test.com/a.png)<!-- width=750 -->` in `action`

#### ~~`metadata` Automatically configure image size (Path mode) `Using this method in inline mode will cause the attachment to fail to upload automatically, so it is abandoned and no image size configuration is performed`~~
- **After turning on this function, attachments cannot be uploaded automatically and need to be uploaded manually. You can turn it on if necessary**
- It is recommended to use the URL mode after uploading the image on the image hosting service
```xml
<!-- $$Macro: \!\[.*\]\((.+)\)\<\!\-\- width-attach=(.*) \-\-\>
     Template: ac:image
     Attachment: ${1}
     Width: ${2} -->
```
- The principle is to add a width tag after the image tag, as shown below
  - `![image1](./test.com/a.png)<!-- width=750 -->`
- **Note**: In action, the width tag will be automatically added after the matching image, and the default width is 750
  - For example, the original content `![image1](./test.com/a.png)` will be replaced by `![image1](./test.com/a.png)<!-- width-attach=750 -->` in `action`

#### `metadata` automatically configures `toc`
```xml
<!-- $$Macro: :TOC:
     Template: ac:toc
     Printable: 'false'
     MinLevel: 2 
     MaxLevel: 4 -->
:TOC:
```

## Warehouse configuration
### Add `workflows`

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
              # print first line in file
              head -n 1 $(echo $file | tr -d '\\')
              # config image width 750
              # sed -i -E 's/(.*\!\[.*\]\(.*\))([ ]*<\!\-\-[ ]*width=[0-9]*[ ]*\-\->)*/\1<!-- width=750 -->/g' $(echo $file | tr -d '\\')
              # path convert
              # sed -i -E 's/(.*\!\[.*\]\([ \t]*([a-zA-Z]\:)?(\.*(\/|\\))?([^:\r\n\t\!]*\.[a-zA-Z]+)[ \t]*\))([ ]*<\!\-\-[ \t]*width=[0-9]*[ \t]*\-\->)*/\1<!-- width-attach=750 -->/g' $(echo $file | tr -d '\\')
              # url convert
              sed -i -E 's/(.*\!\[.*\]\([ \t]*(http(s)?:\/\/)([^:\r\n\t\!]*(\.[a-zA-Z]+)?)[ \t]*\))([ ]*<\!\-\-[ \t]*width=[0-9]*[ \t]*\-\->)*/\1<!-- width=750 -->/g' $(echo $file | tr -d '\\')
              # build and deploy
              mark --ci --debug --trace -p $MARK_PASS -b $MARK_URL -f $(echo $file | tr -d '\\') || exit 1;
            else
              echo "$(echo $file | tr -d '\\') not config metadata info, ignore file"
            fi
          done
```

### add `Actions Secrets`
```yml
ATLASSIAN_USERNAME: username
ATLASSIAN_API_TOKEN: api token
ATLASSIAN_BASE_URL: confluence base url
```

### Test success


## references
- [changed files][changed files]
- [mark][mark]
- [confluence doc][confluence doc]
- [github actions][github actions]

[changed files]: https://github.com/tj-actions/changed-files
[mark]: https://github.com/kovetskiy/mark
[confluence doc]: https://confluence.atlassian.com/doc/code-block-macro-139390.html
[github actions]: https://docs.github.com/zh/actions/about-github-actions/understanding-github-actions
[confluence api token]: https://support.atlassian.com/atlassian-account/docs/manage-api-tokens-for-your-atlassian-account/