# 心动网络 • 公开文档

使用兼容 GitHub 的 Jekyll：[github-pages](https://github.com/github/pages-gem/)

## 添加、更新

- 见 `_posts` 目录中的文件
- 文件名必须以类似 `YYYY-mm-dd-` 格式的日期开头，后面的文字即为文档的URL，如 `2018-08-13-releasing-for-mobile-game.md` 生成的HTML的URL就是 `/docs/releasing-for-mobile-game.html`
- 文档格式为GitHub兼容的Markdown，但是在文件开始需要声明3个必须的Metadata：
    - `layout`：固定为 `post`
    - `title`：文档标题，会显示在页面的 `<title>` 中
    - `date`：设置为最后更新日期，格式为 `YYYY-mm-dd HH:MM:DD +0800`

## 搭建本地预览环境

```bash
# 首先切换到 gh-pages 分支
git checkout gh-pages
# 使用 Ruby 的 bundle 工具安装依赖库
bundle install
# 启动本地环境
jekyll serve -l
```
