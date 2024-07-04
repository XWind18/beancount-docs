# 说明
本项目fork自beancount doc，使用 ChatGPT-4o 进行翻译。
# Beancount 文档

https://beancount.github.io/docs/

源文件位于[docs](docs/)目录中。

这些 Markdown 格式的文档是从[官方 Beancount 文档](http://furius.ca/beancount/doc/index)自动生成的。

您[可以贡献](CONTRIBUTING.md)。

## Beancount Google Doc 转换器

### 安装

转换器需要 Python 3.6 - 3.10。

建议创建 virtualenv：

```bash
python3 -m venv venv
. venv/bin/activate
```

安装依赖项：

```bash
pip install -r requirements.txt
```

### 使用

导出并转换单个文档：

```shell
# 将 Google 文档导出为 docx 文件
python export.py document "100tGcA4blh6KSXPRGCZpUlyxaRUwFHEvnz_k9DyZFn4" doubleentry.docx
# 导出图表
python export.py drawings "100tGcA4blh6KSXPRGCZpUlyxaRUwFHEvnz_k9DyZFn4" drawings
# 将 docx 文件转换为 markdown
python export.py convert doubleentry.docx doubleentry.md --drawing-dir=drawings
```

导出并转换所有文档：

```bash
python crawl.py
```

## 文档网站

使用[MkDocs](https://www.mkdocs.org/)生成静态网站：

```bash
python build.py serve
```

部署到 GitHub Pages：

```bash
python build.py gh-deploy
```
