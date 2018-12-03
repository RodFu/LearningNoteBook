# gitbook安装

------

GitBook 是一个基于 Node.js 的命令行工具，支持 Markdown 和 AsciiDoc 两种语法格式，可以输出 HTML、PDF、eBook 等格式的电子书，使得所记录的知识更便于管理和传播。

## 安装步骤如下：

### 1. 下载安装node.js
> [下载地址: https://nodejs.org/en/download/](https://nodejs.org/en/download/)，如果是下载的非安装包，需要设置Path环境变量

### 2. 安装gitbook工具
> npm install -g gitbook-cli  
> gitbook -V

### 3. 安装 Calibre
> [下载地址： https://calibre-ebook.com/download_windows](https://calibre-ebook.com/download_windows)，执行可执行文件安装完成后，Path环境变量会被自动设置

## 使用步骤如下：

### 1. 进入笔记本根目录，并执行如下命令初始化gitbook
> gitbook init

执行初始化命令后，会在根目录生成如下两个文件
> * README.md —— 笔记的介绍写在这个文件里
> * SUMMARY.md —— 笔记的目录结构在这里配置

### 2. 编辑完成笔记内容后，生成电子版文档
> gitbook init —— 根据最新笔记内容，重新生成必要的目录  
> gitbook pdf ./ ./mybook.pdf —— 生成pdf电子书

当然，还可以生成html格式的电子书
> gitbook build [书籍路径] [输出路径]