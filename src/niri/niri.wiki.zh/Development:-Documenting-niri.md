niri的文档文件位于`docs/wiki/`，应该至少在三个系统中可查看和浏览：

*   GitHub仓库的markdown文件预览
*   [GitHub仓库的wiki](https://github.com/YaLTeR/niri/wiki)
*   [文档网站](https://yalter.github.io/niri/)

## GitHub仓库的wiki

这是通过`.github/workflows/ci.yml`中的`publish-wiki`作业生成的。
为了使此作业在您的fork中按预期运行，您需要在GitHub上的仓库设置中启用wiki功能。
这可能对贡献者有用，以验证wiki是否按您期望的方式生成。

## 文档网站

文档网站使用[mkdocs](https://www.mkdocs.org/)生成。
配置文件位于`docs/`中。

要设置和本地运行文档网站，建议使用[uv](https://docs.astral.sh/uv/)。

### 使用uv在本地提供网站服务

在`docs/`子目录中：

*   `uv sync`
*   `uv run mkdocs serve`

文档网站现在应该在http://127.0.0.1:8000/niri/上可用

在开发服务器运行时对文档进行的更改将导致浏览器中的页面自动刷新。

> \[!TIP]
> 图像可能不可见，因为它们存储在Git LFS上。
> 如果是这种情况，请运行`git lfs pull`。

## 元素

链接、警报、图像和代码片段等元素应该在GitHub上的markdown文件预览、GitHub仓库的wiki和文档网站中按预期工作。

### 链接

链接在所有情况下都应该是相对的（例如`./FAQ.md`），除非它是外部链接。
链接应该有锚点，如果它们旨在将用户引导到页面上的特定部分（例如`./Getting-Started.md#nvidia`）。

> \[!TIP]
> mkdocs将在相对链接指向不存在的文档或不存在的锚点时终止。
> 这意味着构建文档时CI流水线将失败，本地的`mkdocs serve`也是如此。

### 警报

> \[!IMPORTANT]
> 这与其他`mkdocs`-基于文档的区别。

警报，或提示应按[GitHub定义的方式](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax#alerts)编写。
上面的警报是这样写的：

    > [!IMPORTANT]
    > 这与其他`mkdocs`-基于文档的区别。

### 图像

图像应有指向`docs/wiki/img/`中资源的相对链接，并应包含合理的替代文本。

### 视频

为了与mkdocs和GitHub Wiki兼容，视频需要用`<video>`标签包装（由mkdocs显示）并再次将视频链接作为后备文本（由GitHub Wiki显示）并用空行填充。

```html
<video controls src="https://github.com/user-attachments/assets/379a5d1f-acdb-4c11-b36c-e85fd91f0995">

https://github.com/user-attachments/assets/379a5d1f-acdb-4c11-b36c-e85fd91f0995

</video>
```

### 代码片段

配置和代码片段通常应标注语言。

如果代码片段中使用的语言是KDL，请像这样打开代码块：

````md
```kdl
````