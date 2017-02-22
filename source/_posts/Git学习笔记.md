---
title: Git
date: 2017-02-21 11:11:54
tags: Git
---
### Git常用命名

#### 常用

* `init`初始化一个git仓库

* `git add`将文件增加到暂存区
  ```
  git add .
  ```

* `git commit`将暂存区文件提交
  ```
  git commit -m "备注说明"
  ```

* `git log`查看提交记录

* `git diff`

#### 远程仓库

* `git clone git@github.com:michaelliao/gitskills.git`从远程仓库克隆

* `git push origin master`将本地文件推送到指定远程仓库的分支

* `git pull`取最新的远程仓库文件

#### 分支
* `git branch dev`创建分支

* `git checkout dev`切换分支

  ```
  git checkout -b dev
  ```

* `git branch`查看当前分支  
