---
title: 简记Hexo及NexT主题的安装使用
date: 2018-04-08 13:11:08
tags:
categories: [Hexo]
---
#### 官方文档：
[Hexo中文文档][1]
[NexT中文文档][2]
[NexT源码][3]

在Mac系统下搞了个博客，简单的记述一下从无到有的过程。
在官方文档里有详细的安装以及使用教程，不想看文档的话可以按照下面的步骤快速的搭建。

#### 安装 [Homebrew][16]
```
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
#### 安装 [Git][17]
```
$ brew install git
```

#### 安装 [nvm][18]
nvm: Node Version Manager，安装 Node.js 的最佳方式是使用 nvm。
```
$ curl https://raw.github.com/creationix/nvm/master/install.sh | sh
// 或者
$ curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.9/install.sh | bash
```
安装完成后，重启终端nvm环境才生效。

#### 安装 [Node.js][19]
```
$ nvm install stable
```

#### 安装 [Hexo][20]
```
$ npm install -g hexo-cli
```
hexo 简单的命令格式，比如：
hexo g == hexo generate
hexo d == hexo deploy
hexo s == hexo server
hexo n == hexo new

#### 创建博客
```$ hexo init <folder>
$ cd <folder>
$ npm install
$ hexo n "我的新博客"
$ hexo g -d
$ hexo s
```
浏览器输入：http://localhost:4000 就可以看到了

Hexo目录：
_config.yml  博客的配置文件
scaffolds      博客文章模板
source          博客文章目录
themes        存放主题文件

>**Tip: ** hexo deploy出错解决方法：

```
$ npm install hexo-deployer-git --save
```

#### 安装 [NexT][3] 主题
```
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```

**配置主题**
打开站点配置文件**_config.yml**，找到**theme**字段，并将其值更改为**next**
```
theme: next
```

**更改主题外观**
找到 .../themes/next/**_config.yml** 文件，更改 **scheme** 字段
```
# Schemes
# scheme: Muse
scheme: Mist
# scheme: Pisces
# scheme: Gemini
```

**使主题生效：**
```
$ hexo clean
$ hexo g -d
$ hexo s
```

**Hexo发布博客引用自带图片的方法**
1, 修改 _config.yml 配置文件 post_asset_folder 项为 true。在当前blog文件的的同级目录有个跟文件名相同的文件夹，把图片放进去就可以了，

2, 安装 [hexo image asset 插件][HexoImageAsset插件]
```
npm install hexo-asset-image --save
```

3， Just use \!\[logo\]\(logo.jpg\) to insert logo.jpg.

到这里博客的本地搭建已经完成了，这只是开始。
如果想要博客让别人也能看见，那就得申请域名把博客部署到服务器，然后别人通过域名访问就可以浏览你的博客啦。
当然你可以把博客部署到[GitHub][11]或者[Coding][12]又或者别的服务器等，域名购买有[DNSPod][13]，[万网][14]，[GoDaddy][15]等等，有国内，国外自己考虑，之后还想要自己的博客有个性或者更好用那就选一个好看的主题DIY了。

至于写Markdown写作工具，各有所爱，我用的是[Sublime Text][21]，装上 [Package Control][22]，[Markdown​Editing][23],  [Markdown Preview][24]或者[Omni​Markup​Previewer][25] 等插件，就可以愉快的玩耍了。

#### Sublime Markdown

**MAC 键盘**
⌘：command
⌥：option/alt
⇧：shift
⌃：control
⎋：esc

**MarkdownEditing:**
- 安装后针对 md\mdown\mmd\txt 格式文件启用插件。颜色方案仿 Byword 及 iA writer。
- 自动匹配星号（*）、下划线（_）及反引号（`），选中文本按下以上符号能自动在所选文本前后添加配对的符号，方便粗体、斜体和代码框的输入。
- 直接输入配对的符号后按下退格键（backspace），则两个符号都会被删除；直接输入配对的符号后按下空格键，则会自动删除后一个。
- 对“选中文字后输入左括号”这一动作进行了调整，以便插入 markdown 链接。
- 拷贝一个链接，选中文本后按下 ⌘⌥V 会自动插入内联链接。
- 拷贝一个链接，选中文本后按下 ⌘⌥R 会自动插入引用链接。
- ⌘K 插入链接；⌘⇧K 插入图片。
- ⌘B 和 ⌘I 分别用于加粗体和斜体。
- 选中文本后按下 # 会自动在文本前后进行配对，可重复按下来定义标题级别，还可用 ⌘⇧空格来增加 # 与所选文本之间的空格（也是自动配对的）。

**OmniMarkupPreviewer:**
- Command +Option +O: 在浏览器中预览
- Command+Option+X: 导出HTML
- Ctrl+Alt+C: HTML标记拷贝至剪贴板

[OmniMarkupPreviewer 404][Omnimarkuppreviewer404]

以下列出的条目可供参考，或许能帮上你～

#### 部署本地文件到github及源代码托管参考：
[Hexo+GitHub Pages搭建的个人博客][4] 
[Mac搭建hexo博客][5]

#### 域名绑定参考 ：
[hexo边搭边记][6]
[在github上搭建自己的主页和顶级域名的绑定][7]

#### NexT主题定制参考：
[hexo框架基于next主题定制][8]
[基于Hexo+Next主题的个人博客搭建定制优化][9]   
[hexo的next主题个性化教程:打造炫酷网站][10] 

#### Markdown 语法参考 ：
[Markdown 语法整理大集合2017][26]

[1]: https://hexo.io/zh-cn/
[2]: http://theme-next.iissnan.com/getting-started.html
[3]: https://github.com/iissnan/hexo-theme-next
[4]: http://www.aisun.org/2017/09/hexo+github+pages/index.html
[5]: https://depthlove.github.io/2015/06/12/use-hexo-create-blog-in-mac/index.html
[6]: http://blog.sunnyxx.com/2014/02/27/hexo_startup/
[7]: https://blog.csdn.net/tyro_java/article/details/51348477
[8]: http://www.aisun.org/2017/10/hexo-next+dingzhi/index.html
[9]: https://blog.csdn.net/miaoqiucheng/article/details/72794165
[10]: http://shenzekun.cn/hexo%E7%9A%84next%E4%B8%BB%E9%A2%98%E4%B8%AA%E6%80%A7%E5%8C%96%E9%85%8D%E7%BD%AE%E6%95%99%E7%A8%8B.html
[11]: https://github.com/
[12]: https://coding.net/
[13]: https://www.dnspod.cn/
[14]: https://wanwang.aliyun.com/
[15]: https://sg.godaddy.com/
[16]: https://brew.sh/
[17]: https://git-scm.com/
[18]: https://github.com/creationix/nvm
[19]: https://nodejs.org/en/
[20]: https://hexo.io/
[21]: https://www.sublimetext.com/
[22]: https://packagecontrol.io/
[23]: https://packagecontrol.io/packages/MarkdownEditing
[24]: https://packagecontrol.io/packages/Markdown%20Preview
[25]: https://packagecontrol.io/packages/OmniMarkupPreviewer
[26]: https://www.jianshu.com/p/b03a8d7b1719
[Omnimarkuppreviewer404]: https://stackoverflow.com/questions/35798823/omnimarkuppreviewer-404
[HexoImageAsset插件]: https://github.com/CodeFalling/hexo-asset-image
