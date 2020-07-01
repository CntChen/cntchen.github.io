## 部署
使用 hexo 生成静态博客。
如果是首次搭建自己的博客，参考 hexo 的官方教程搭建。

搭建后，一个需要解决的问题是：如何跨机器保留博客源码。
建立两个分支，`hexo`和`master`，其中：
`hexo` 是页面源码分支
`master` 是生成后的静态页面分支

### 新机器使用
* clone
```
$ git clone https://github.com/CntChen/cntchen.github.io.git
```

* download theme, what I am using is [`next`](https://github.com/iissnan/hexo-theme-next)
```
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```

* change branch
```
$ git checkout hexo
```

* new post
```
$ hexo new [post] "new blog title"
```

* commit
```
$ git add .
$ git commit -m "say something"
```

* push
```
// 提交到 hexo 分支
$ git push origin hexo
```

* commit blog
```
$ hexo g -d
```
