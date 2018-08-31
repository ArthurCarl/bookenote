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
