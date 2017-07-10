#### PostgREST 文档的 Sphinx 源文件 

生成 HTML 版本:

1. 通过 [sphinx website](http://sphinx-doc.org/latest/install.html) 安装 Sphinx
2. git clone repo
4. 生成 HTML
    ```bash
    cd postgrest-docs
    sphinx-build -b html -a -n . _build

    # 在浏览器中 _build/index.html
    ```

---

**Sphinx 安装提示:**

* 如果是 OSX 你可能想直接通过 homebrew 来安装 Python from homebrew - 那么你可以简单的通过 `pip install sphinx` 来安装.
* 为了更快的在本地预览，可以试试 [sphinx-autobuild](https://github.com/GaretJax/sphinx-autobuild).
