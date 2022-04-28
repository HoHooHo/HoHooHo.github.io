# Windows 部署 mkdocs

## 安装 Python

使用 python3.x 版本，确保勾选了 安装 pip

## 安装 mkdocs

cmd 执行命令

```
pip install mkdocs
```

## 安装 mkdocs-bootswatch 主题

cmd 执行命令

```
pip install mkdocs-bootswatch
```

## 安装 markdown_katex 数学函数渲染库

cmd 执行命令

```
pip install markdown_katex

pip install python-markdown-math
```

## 启动

cmd 执行命令

```
mkdocs new 
mkdocs serve
mkdocs gh-deploy
```


# 部署到 Github




    - markdown_katex:
        no_inline_svg: True
        insert_fonts_css: True
    - admonition