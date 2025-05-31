---
title: 用Hexo+Netlify免费部署自己的博客
date: 2025-01-18 18:03:40
categories: 
- 技术
excerpt: 本文介绍如何使用 Hexo 和 Netlify 免费部署个人博客，包括安装 Hexo、配置 GitHub 仓库、部署到 Netlify 以及自定义域名的步骤。
tags:
- 博客
- Hexo
---

最近看到周围的朋友都搞了自己的博客，非常羡慕。然后上网搜了一下，发现了这么一个好东西：

**Hexo**：[hexo.io](https://hexo.io "点我前往Hexo官网")

Hexo 是一款基于 Node.js 的博客框架，上手快，简洁高效，使用 Markdown 来解析文章。

## 准备工作
你需要安装以下软件包： ``nodejs`` ``git``

## 安装
首先安装 hexo cli

```bash
npm i -g hexo-cli
```
如果下载太慢可以切换下载源
```bash
# 该命令会把 npm 的下载源切换成国内的淘宝源
npm config set registry https://registry.npmmirror.com

# 该命令会把 npm 的下载源切换回官方的源
npm config set registry https://registry.npmjs.org
```

# 开始使用
使用该命令创建你的博客目录
```bash
hexo init {目录名}
```
如输入 ``hexo init blog`` ，那么就会在你的当前目录下创建一个 blog 文件夹，里面包含了你的博客所需要的文件。

接下来使用该命令进入目录。
```bash
cd blog
```
然后安装所需依赖。
```bash
npm install
```
一个 Hexo 的基本目录就安装完成了。

接下来打开 ``package.json`` 文件，把 ``scripts`` 那一段改成下面这段
```bash
"scripts": {
  "build": "hexo generate",
  "clean": "hexo clean",
  "server": "hexo server",
  "netlify": "npm run clean && npm run build"
}，
```
保存并退出。

然后你会看到有一个名为 ``_config.yml`` 的文件，这就是你博客的配置文件。

配置详情可以在[这里](https://hexo.io/zh-cn/docs/configuration.html "点我前往")查看。

配置好之后运行这个命令
```bash
hexo s
```
这将会在本地的 4000 端口上开启你的博客服务器，在本机的浏览器中输入 http://localhost:4000/ 即可查看。

# 发布你的文章

使用以下命令来发布你的第一篇文章。
```bash
hexo new post "First"
```
这将会在 source/_posts/ 目录下生成文件 ‘First.md’，打开编辑器进行编辑，使用 Markdown 语法。

你会看到里面有类似这样子的内容：
```yaml
---
title: 用Hexo+Netlify免费部署自己的博客
date: 2024-02-24 13:08:49
categories: 
- 技术
tags:
- 博客
- Hexo
---

```
这些就相当于你博客的元数据，``title`` 和 ``date`` 不用多说，``categories`` 指的是你文章的分类，``tags`` 则是这篇文章的标签。

其中 ``categories`` 和 ``tags`` 不是必须的，可以不填。

编辑好之后就可以用这个命令将它发布。
```bash
hexo g
```
然后就可以使用 ``hexo s`` 看看本地效果了。

# 上传至 Github
打开 Github 并注册帐号，这里就不多赘述。

接下来我们新建一个仓库。先点击右上角个人头像，再选择 ``Your respositories`` ,选择 ``New`` 。

接下来如图所示
<br>![](/img/2025-01-19T18-44-56.208Z.png)

然后以 ssh 的方式将其 clone 下来

<br>![](/img/2025-01-19T18-53-15.796Z.png)

方法：
```bash
cd .. //回到博客外面的文件夹
git clone git@github.com:/用户名/仓库名.git
```
**一定要用SSH ！！！**

如果出现无权限的问题请参考[这里](https://zhuanlan.zhihu.com/p/62022220 "解决无权限问题")为你的 Github 添加 ssh 密钥。

clone 下来后，我们发现仓库是空的，这时候我们再把博客文件夹中的所有内容复制过去，然后我们需要提交这个博客仓库
```bash
git add .
git commit -m "写入备注"
git push
```
这样子就上传至 Git 仓库了。

# 部署到 Netlify

[Netlify官网](https://www.netlify.com)

直接用自己的电子邮箱注册一个帐号，然后来到主页，新建一个 Site。

![](https://pic.imgdb.cn/item/65da23619f345e8d03ed0897.jpg)

从 Github 导入。

![](https://pic.imgdb.cn/item/65da29a49f345e8d03fc54ff.jpg)

选择你的仓库。

``Site name`` 就是作为你博客的子域名。我设置了 ``sudabaka`` ，那么 Netlify 分配给我的就是 https://sudabaka.netlify.app

这里需要这样填

![](https://pic.imgdb.cn/item/65da2fad9f345e8d030966fb.jpg)

一切准备就绪，就按下底部的 Deploy 按钮，然后等待部署完成，完成后我们就可以通过 Netlify 的 URL 访问了！

# 自定义域名（可选）

Netlify 允许你使用自己的域名，首先我们打开侧边的域名管理。

![](https://pic.imgdb.cn/item/65da31e59f345e8d030d787f.jpg)

添加你自己的子域名( Add a domain )，这时候它会同时添加主域名，并且要求你使用它的 DNS ，如果你这个域名有服务器的话，那我建议不要将子域名设置为 `www` ，因为它会自动重定向到主域名，导致你的博客无法访问，我的建议是使用 blog 作为子域名。

然后我们给域名添加 CNAME 记录，指向 Netlify 的域名，接下来的 HTTPS 我们只需要用[ Let's Encrypt ](https://letsencrypt.org/zh-cn/)签发一个证书然后手动安装就可以了。如果需要发帖子的话，就像前面说的一样。
```bash
hexo new post "帖子名"
//编辑后
hexo g
git add .
git commit -m "更新内容"
git push
```
push 之后 Netlify 那边就会自己部署了。

# 写在最后

这是我第一次写博客，可能有些不足之处，还是希望大家多多关照qwp。