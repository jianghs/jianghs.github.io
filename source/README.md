# 我的个人博客 Life Line

旨在记录自己的生活轨迹，包括但不限于读书笔记、代码心得、生活感受......，就如同自己的生命线。

## 写作

```git
hexo new "文章名"
```

通过该命令即可新建一遍文章，对应的会在 source/_posts 文件夹下新增一个 markdown 文件，使用 VS CODE 进行编辑即可。

## 发布

```git
git add *
git commint -m "注释"
git push
```

集成 travis 这个轻量的 CI 工具，每次将文件 push 到 github 远程仓库时， travis 会自动构建博客并发布。
