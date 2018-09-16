---
title: Hexo
date: 2018-09-16 21:24:48
categories: others
tags: Blog
---

Hexo折腾记录
<!--more-->

## Hexo基本操作

- 新建文章
  `hexo new "My New Post`
- 跑本地服务 localhost:4000
  `hexo server -s`
- 生成网站静态文件
  `hexo`
- 部署生成的文件到github
  `hexo d`
- 新建页面
  `hexo new page "My new page"`
- 清除缓存文件
  `hexo clean`
- 删除文件
  直接删除本地文章，然后`hexo clean` && `hexo g` && `hexo d`即可

## 使用github备份博客
### 添加过滤内容

#### 博客根目录

```
.deploy_git     网站静态文件(git)
node_modules        插件
public          网站静态文件
scaffolds       文章模板
source          博文等
themes          主题
_config.yml     网站配置文件
package.json        Hexo信息
db.json
```

#### 添加过滤内容

实际备份中，`.deploy_git`、`public`文件夹和我们的`博客`分支内容重复，所以略过。而`_config.yml`为我们的站点配置文件，安全起见，这个文件也不备份。  所以，我们在根目录下面建一个`.gitignore`文件来建立“黑名单” 

文件内容：

```
/.deploy_git
/public
/_config.yml
```

(然而这里我对hexo不熟悉，所以没有建立黑名单,以后有机会再弄吧)

### 另外一种选择
在github上只备份博客文章即可，而我选择这种。

## 资源文件夹

资源（Asset）代表 `source` 文件夹中除了文章以外的所有文件，例如图片、CSS、JS 文件等。比方说，如果你的Hexo项目中只有少量图片，那最简单的方法就是将它们放在 `source/images` 文件夹中。然后通过类似于 `![](/images/image.jpg)` 的方法访问它们。

### 文章资源文件夹(个人选择这种方式)

对于那些想要更有规律地提供图片和其他资源以及想要将他们的资源分布在各个文章上的人来说，Hexo也提供了更组织化的方式来管理资源。这个稍微有些复杂但是管理资源非常方便的功能可以通过将 `config.yml`文件中的 `post_asset_folder` 选项设为 `true` 来打开。

```
_config.ymlpost_asset_folder: true
```

当资源文件管理功能打开后，Hexo将会在你每一次通过 `hexo new [layout] <title>` 命令创建新文章时自动创建一个文件夹。这个资源文件夹将会有与这个 markdown 文件一样的名字。将所有与你的文章有关的资源放在这个关联文件夹中之后，你可以通过相对路径来引用它们，这样你就得到了一个更简单而且方便得多的工作流。

### 相对路径引用的标签插件

通过常规的 markdown 语法和相对路径来引用图片和其它资源可能会导致它们在存档页或者主页上显示不正确。在Hexo 2时代，社区创建了很多插件来解决这个问题。但是，随着Hexo 3 的发布，许多新的标签插件被加入到了核心代码中。这使得你可以更简单地在文章中引用你的资源。

```
{% asset_path slug %}
{% asset_img slug [title] %}
{% asset_link slug [title] %}
```

比如说：当你打开文章资源文件夹功能后，你把一个 `example.jpg` 图片放在了你的资源文件夹中，如果通过使用相对路径的常规 markdown 语法 `![](/example.jpg)` ，它将 *不会* 出现在首页上。（但是它会在文章中按你期待的方式工作）

正确的引用图片方式是使用下列的标签插件而不是 markdown ：

```
{% asset_img example.jpg This is an example image %}
```

通过这种方式，图片将会同时出现在文章和主页以及归档页中。

### 修改网站logo
- 需要`32*32` 的ico图片并改名为`w-32x32.ico`,放在`/themes/next/source/images`下

- 主题配置文件:

  ```yml
  favicon:
    small: /images/w-16x16.ico
    medium: /images/w-32x32.ico
    apple_touch_icon: /images/w.png
    safari_pinned_tab: /images/w.svg
  ```
