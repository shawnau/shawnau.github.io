# git 从fork的项目更新自己的项目的简易手段


RT

<!--more-->

 - 假设白井黑子从`[misaki/railgun.git]`处fork了炮姐的项目到自己的项目`[kuroko/railgun.git]`
 - 过了段时间, 炮姐已经更新了好几个版本黑子才想起来, 于是她想把自己fork的项目也同步更新, 简单的解决办法:

```bash
# 先把自己的项目clone到本地
git clone kuroko/railgun.git
# 再创建一个upstream分支, 指向炮姐的项目
git remote add upstream misaki/railgun.git
# 从炮姐那取得项目更新
git fetch upstream
# 和本地的master分支合并
git merge upstream/master
```

Refer: https://help.github.com/articles/fork-a-repo/
