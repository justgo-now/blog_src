---
title: GitHub 如何向开源项目提PR
date: 2019-07-29 13:16:19
tags: [GitHub]
---

- fork 到自己的仓库
- git clone 到本地
- 上游建立连接
  `git remote add upstream 开源项目地址`
- 创建开发分支 (非必须)
  `git checkout -b dev`
- 修改提交代码
  `git status` `git add .` `git commit -m` `git push origin branch`
- 同步代码三部曲
  `git fetch upstream` `git rebase upstream/master` `git push origin master`
- 提交pr
  去自己github仓库对应fork的项目下new pull request