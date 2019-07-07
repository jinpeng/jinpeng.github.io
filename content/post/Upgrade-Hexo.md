---
title: Upgrade Hexo
date: 2018-04-24 13:33:03
tags:
---
## Why Upgrade Hexo
在使用hexo和github pages的过程中，经常出现hexo-cli报错，很早就想重新做一下开发环境，甚至想要换成其他的博客生成器，比如：Hugo。然而，另起炉灶总是要花费更多的时间，所以决定还是升级一下Hexo。
之前尝试重做环境遇到了很多坑，因此决定另选吉地，重新开始。

## Let's Do It

首先，升级hexo：

```
$ npm i hexo-cli -g
$ hexo version
hexo: 3.7.1
hexo-cli: 1.1.0
os: Darwin 17.5.0 darwin x64
http_parser: 2.8.0
node: 9.10.1
v8: 6.2.414.46-node.23
uv: 1.19.2
zlib: 1.2.11
ares: 1.13.0
modules: 59
nghttp2: 1.29.0
napi: 2
openssl: 1.0.2o
icu: 61.1
unicode: 10.0
cldr: 33.0
tz: 2018c
```

其次，在新的目录下创建blog site：

```
$ hexo init jinpeng.github.io
$ cd jinpeng.github.io
$ npm i
$ hexo s
```

打开浏览器访问：http://localhost:4000/，即可看到新建的博客。

第三，拷贝原来的博客源文件到新博客目录下：

```
$ cp ~/old/jinpeng.github.io/source/_posts/*.md source/_posts/
```
第四，编辑_config.yml，设置相应的参数。之所以不拷贝原来的_config.yml，是为了避免新旧版本不兼容。

```
title: Jinpeng's Blog
subtitle:
description:
keywords: 
author: Dong Jinpeng
language: zh-cn
timezone: Asia/Shanghai

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://jinpeng.github.io

```
注意，其中的language是zh-cn，而不是zh-CN，这主要是使用的theme tranquilpeak的语言是小写。
这时候，运行hexo s，就可以看到之前的posts了，但还是用的缺省的landscape。

第五，更换theme，编辑_config.yml

```
theme: tranquilpeak
```
下载theme包：

```
$ wget https://github.com/LouisBarranqueiro/hexo-theme-tranquilpeak/releases/download/v2.0.0/hexo-theme-tranquilpeak-built-for-production-2.0.0.zip
$ unzip hexo-theme-tranquilpeak-built-for-production-2.0.0.zip
$ mv hexo-theme-tranquilpeak-built-for-production-2.0.0 themes/tranquilpeak
```

创建all-categories，all-tags，all-archives等页面：

```
$ hexo new page "all-categories"
$ hexo new page "all-tags"
$ hexo new page "all-archives"
```

编辑生成的index.md，分别设置为以下内容：

source/all-categories/index.md

```
---
title: "all-categories"
layout: "all-categories"
comments: false
---
```

source/all-tags/index.md

```
---
title: "all-tags"
layout: "all-tags"
comments: false
---
```

source/all-archives/index.md

```
---
title: "all-archives"
layout: "all-archives"
comments: false
---
```

增加搜索页面，使用Algolia：

```
$ npm install hexo-algoliasearch --save
```

编辑_config.yml，增加Algolia配置：

```
# Search
algolia:
  appId: "xxxxxxxx"
  apiKey: "xxxxxxxxxxxxxxxxxxxxxxxx"
  adminApiKey: "xxxxxxxxxxxxxxxxxxxxxxxxx"
  chunkSize: 5000
  indexName: "jp-hexo-blog"
  fields:
    - content:strip:truncate,0,500
    - excerpt:strip
    - gallery
    - permalink
    - photos
    - slug
    - tags
    - title
```

其中的appId，apiKey和adminApiKey需要到[Algolia网站](https://www.algolia.com/)去注册帐号申请，帖子不多、访问量不大可以使用免费的。

建索引需要手动执行：

```
$ hexo algolia
```

第六，允许使用数学公式，MathJax。由于缺省的Markdown渲染引擎不兼容，需要换用Kramed，并且做简单的修改，hexo-math也需要换成hexo-renderer-mathjax。

```
$ npm uninstall hexo-renderer-marked --save
$ npm install hexo-renderer-kramed --save
$ npm uninstall hexo-math --save
$ npm install hexo-renderer-mathjax --save
```

编辑node_modules/hexo-renderer-kramed/lib/renderer.js，把下面的代码：

```
// Change inline math rule
functionformatText(text) {
    // Fit kramed's rule: $$ + \1 + $$
    return text.replace(/`\$(.*?)\$`/g, '$$$$$1$$$$');
}
```

改为：

```
// Change inline math rule
function formatText(text) {
    return text;
}
```

编辑node_modules/hexo-renderer-mathjax/mathjax.html，替换CDN URL为MathJax官网上的CDN地址：

```
<script src='https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-MML-AM_CHTML' async></script>
```

第七，部署到github pages：

编辑_config.yml，修改deploy参数

```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://github.com/jinpeng/jinpeng.github.io.git
  branch: master
  message: 
```

执行部署命令：

```
$ hexo deploy
```

打开浏览器访问：http://jinpeng.github.io/就可以看到新部署的博客。

第八，替换github上原有的代码。原来的代码保存在hexo分支，master分支用于保存部署的页面。

```
$ git init
$ git add .
$ git commit -m "upgrade hexo"
$ git remote add origin https://github.com/jinpeng/jinpeng.github.io.git
$ git push origin :hexo
$ git branch hexo
$ git checkout hexo
$ git push origin hexo
```

其中git push origin :hexo命令删除了远程仓库里的hexo分支。


