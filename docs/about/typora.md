### 简介

Typora是一款轻量级的Markdown编辑器，在这里汇总一些有用的手动激活的办法！

### 方法一：修改LicenseIndex配置文件

1. 安装软件typora
2. 安装完后，进入typora的安装目录下的 \resources\page-dist\static\js 目录，找到 LicenseIndex开头的文件

![avatar](https://raw.githubusercontent.com/cherryxiu/image/master/mkdocs/typora_1.png)

3. 用文本编辑器打开该文件，找到`hasActivated="true"==e.hasActivated`代码，并将其替换为`hasActivated="true"=="true"`（有的文件可能是三个等号，所以直接搜`hasActivated`比较好）

![avatar](https://raw.githubusercontent.com/cherryxiu/image/master/mkdocs/typora_2.png)
