---
title: hexo_command
tags:
---

### 文章

```bash
$ hexo new "new article"
```

### 草稿

草稿相当于很多博客都有的“私密文章”功能。
```bash
$ hexo new draft "new draft"
```
会在source/\_drafts目录下生成一个new-draft.md文件。但是这个文件不被显示在页面上，链接也访问不到。也就是说如果你想把某一篇文章移除显示，又不舍得删除，可以把它移动到\_drafts目录之中。

如果你希望强行预览草稿，更改配置文件：
```bash
$ render_drafts: true
```

或者，如下方式启动server：
```bash
$ hexo server --drafts
```

下面这条命令可以把草稿变成文章，或者页面：
```bash
$ hexo publish [layout] <filename>
```
