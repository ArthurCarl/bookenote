# Atom

### Getting Started

### Atom Basics

`Ctrl+Shift+P` 命令快捷键

`Ctrl+,` 设置面板

`Ctrl+O` 打开文件

`Ctrl+S` 保存

`Ctrl+Shift+S` 另存为

`Ctrl+Shift+A` 添加项目文件

`Ctrl+\` 文件树的 toggle

`Alt+\` 光标在文件树和编辑器之间切换

`Ctrl+T` 或者 `Ctrl+P` 打开模糊搜索文件框

`Ctrl+B` 搜索打开的文件

`Ctrl+Shift+B` 上次git提交之后的文件的变化

## Using Atom

### Atom Packages

`apm install <package_name>` 安装包最新版本

`apm install <package_name>@<package_version>` 安装特定版本的包

`apm search <package_name>` 搜索包

`apm view <package_name>` 查看包的简单信息

### Moving in Atom

`Ctrl+Left` 词的最左

`Ctrl+Right` 词的最右

`Home` 行首

`End` 行尾

`Ctrl+Home` 文件头

`Ctrl+End` 文件尾

`Ctrl+G` 跳转到指定的行和列

可将命令调用板中的命令绑定快捷键:`keymap.cson`
```json
'atom-text-editor':
  `ctr-shift-e`:`editor:select-to-previous-word-boundary`
```

`Ctrl+R` symbol导航

`Ctrl+Shift+R` tag导航

`Alt+Ctrl+F2` 书签

`F2` 下一个书签

`Shift+F2` 循环

`Ctrl+F2` 查看书签列表


### Atom Selections

`Shift+Up` 向上选取一行

`Shift+Down` 向下选取一行

`Shift+Left` 向左选取一个字符

`Shift+Right` 向右选取一个字符

`Ctrl+Shift+Left` 选取词的开始

`Ctrl+Shift+Right` 选取词的结束

`Shift+End` 选择到行尾

`Shift+Home` 选择到行头

`Ctrl+Shift+Home` 选取到文件顶部

`Ctrl+Shift+End` 选取到文件底部

`Ctrl+A` 全选

`Ctrl+L` 整行

### Editing and Deleting Text

`Ctrl+J` 将下行添加到本行尾部

`Ctrl+Up/Down` 整行上移或下移

`Ctrl+Shift+D` 复制当前行

`Ctrl+K` `Ctrl+U` 转为大写(组合按)

`Ctrl+K` `Ctrl+L` 转为小写(组合按)

`Alt+Ctrl+Q` 单行过长的格式化为多行

`Ctrl+Shift+K` 删除当前行

`Ctrl+Backspace` 删除光标前一个单词

`Ctrl+Delete` 删除光标后一个单词


`Ctrl+Click` 在单击的位置添加新光标

`Alt+Ctrl+Up/Down` 在当前光标上方/下方添加另一个光标

`Ctrl+D` 选择文档中与当前所选单词相同的下一个单词

`Alt+F3` 选择文档中与当前所选单词相同的所有单词

`Ctrl+M` 跳转到与光标相邻的支架。当没有相邻支架时，它会跳到最近的封闭支架

`Alt+Ctrl+,` 选择当前括号内的所有文本

`Alt+Ctrl+.` 关闭当前的XML/HTML标记

`Ctrl+Shift+U` 切换菜单以更改文件编码

### Find and Replace

`Ctrl+F` 在缓冲区内搜索

`Ctrl+Shift+F` 搜索整个项目

### Snippets -代码片段

创建自己的代码片段

`%USERPROFILE%\.atom` 中的 `snippets.cson` 文件

格式
```
'.source.js':
  'console.log':
    'prefix': 'log'
    'body': 'console.log(${1:"crash"});$2'
```

代码片段和 `source.java` 等文件相关

### Autocomplete
在编辑器中使用 `Tab` 或者 `Enter` 能够查看和插入一些自动补全

### Fold -折叠
`Alt+Ctrl+[` 折叠

`Alt+Ctrl+]` 展开

`Alt+Ctrl+Shift+[` 折叠一切

`Alt+Ctrl+Shift+]` 展开一切

`Ctrl+K` `Ctrl+0-9` 特定缩进级别折叠

`Alt+Ctrl+F` 键入或选择“折叠选择”来折叠代码或文本的任意部分

### Panes
`Ctrl+K` `Up/Down/Left/Right` 分窗

`Ctrl+K` `Ctrl+Up/Down/Left/Right` 焦点移动

`Ctrl+W` 关闭窗口

### Grammer

`Ctrl+Shift+L` 语法选择器

### Atom中的版本控制
