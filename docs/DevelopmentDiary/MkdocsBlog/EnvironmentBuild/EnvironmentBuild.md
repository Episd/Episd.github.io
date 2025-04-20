## 环境搭建

大二从波波学长那儿了解到了 MkDocs 和 Material for MkDocs，觉得很有意思，故尝试自己用 Material for MkDocs 搭建一个个人博客，记录自己的学习。一定要坚持下去啊！！

我希望将博客交给 GitHub 托管，但是我也不知道从本地提交到网页的具体的逻辑是什么。当初陈一波学长告诉我可以用 Git 来提交，那我第一步就是安装 Git。安装时参考了 [Git 详细安装教程（详解 Git 安装过程的每一个步骤）_git安装-CSDN博客](https://blog.csdn.net/mukes/article/details/115693833) 

安装网页是这个：[Git](https://git-scm.com/)

只不过默认编辑器选择了 VsCode，我本身比较喜欢，而且功能强大。

之后打算认认真真了解一下 Git 到底是什么东西。也给自己留一个任务：

- [ ] 了解 Git，并且书写学习笔记

首先先把项目建立好，之后再往里面慢慢写东西，不要害羞，不要觉得自己能力不行，写出来的东西很烂就不想展示。我决定把这个项目从零开始每一点的添加都提交到 GitHub。

接下来就考虑如何建立 Mkdocs项目，参考了这两篇文档：

[Getting Started - MkDocs](https://www.mkdocs.org/getting-started/)

[Installation - Material for MkDocs](https://squidfunk.github.io/mkdocs-material/getting-started/)

在重复阅读文档的时候，浏览器只能通过添加到收藏夹或者从历史记录点击来回到之前看到的位置。这个让我有点点难过，但是在学习的过程中，过于关心这个我觉得并不合适。

我观看下方链接教程中创建 Material for MkDocs 项目时，用命令行创建了 python 虚拟环境。我之前一直是用 Pycharm 的 GUI 界面选择 / 创建虚拟环境的，感觉这个比较有意思：

```python title="创建&启动虚拟环境的相关命令"
# 在工作区的终端输入以下命令
python -m venv name # name 为虚拟环境的文件夹名称
# python -m venv .venv
.\name\Scripts\activate # (1)
# .\.venv\Scripts\activate
```

1. 输入这条语句时可能会报错，一个原因是由于windows的终端处于安全考虑默认不允许运行本地脚本，在终端中先运行`Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned`即可

然后按照教程，在虚拟环境里用 pip 安装 Material for MkDocs，并无异常。

之后我将项目部署到github，也是按照视频的教程。部署之后遇到两个问题：

- 本地运行的时候，点击导航栏的某个索引（MyBatis），网页会跳转到 MyBatis/，但是 push 到 github 上之后，网页却跳转到了 MyBatis.md 导致404
- 项目代码块语法没有正常高亮

第一个问题我通过 AI，了解到问题的原因可能是：我仓库的 **GitHub Pages 设置里，启用了 Jekyll 的默认处理逻辑**，它会错误地解析 `.md` 链接，**把本来跳转到 `/xxx/` 的页面改成了 `/xxx.md`**。

于是在我的项目根目录中新建一个名为`.nojekyll`的空文件，重新 push 到 github 上，就成功了。

第二个问题，经过多次测试，结果居然是我那三行代码的高亮的样式就是全黑的，没有其他颜色。

之后就可以根据文档书写我的博客了