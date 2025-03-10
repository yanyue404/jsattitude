## 目录

- [0\. 注意事项](#%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9)
- [1\. 我们的分支模型](#%E6%88%91%E4%BB%AC%E7%9A%84%E5%88%86%E6%94%AF%E6%A8%A1%E5%9E%8B)
- [2\. 开发场景](#%E5%BC%80%E5%8F%91%E5%9C%BA%E6%99%AF)
- [3\. 发布的流程](#%E5%8F%91%E5%B8%83%E7%9A%84%E6%B5%81%E7%A8%8B)

## 注意事项

1、pull 配置

```shell
# 所有的 pull 命令都按 rebase 的方式执行
git config --global pull.rebase true
```

2、merge 配置

需配置如下：

```shell
# 创建自定义 merge driver
git config --global merge.ours.driver true
```

why？

uat 分支上的 project.config.json、fetch.js、app.js 与 prod 分支上的这几个配置性文件不一致。如果直接 执行 merge 就会使用 uat 分支替换所有 prod 的文件。

而做出如上配置后（加上已有的 `.gitattributes`忽略项）再进行 merge uat 分支上的 project.config.json、fetch.js、app.js 就不会被合并了，prod 会保持自己原先的配置不变。

```shell
# .gitattributes 文件（位于项目根目录）
project.config.json merge=ours
fetch.js merge=ours
app.js merge=ours
```

> 注意： 只能 master 合并其他分支时忽略其他分支上的文件, 其他分支合并 master 无法忽略 master 上的文件，所以这样的操作只能是单向有效。 （master 为默认主分支，即我们项目里的 prod）

## 我们的分支模型

**主线分支**

- `prod`: 产品分支，对应产品服务器环境 (master)
- `uat`： 测试分支，对应测试服务器环境 (dev)

uat 与 prod 为两个的独立程序，uat 为测试版本，prod 为正式版。不能直接在两个分支进行开发，只能通过分支合并的方式并入。

uat 分支可以只能通过 develop-xxx 与 hotfix-xx_xx 分支并入。  
prod 分支可以通过 uat 与 hotfix-xx_xx 分支并入，发布的时候只能使用 uat 分支合并。

**开发分支**

- `develop-yue`：个人开发分支（仅服务于 uat）
- `develop-joe`: 个人开发分支（仅服务于 uat）
- `devlop-li`: 个人开发分支（仅服务于 uat）

个人开发分支应保证与 uat 分支同步并且只能优于 uat，个人开发分支开发完成 push 后，以 rebase 的方式 merge 到 uat（可以使得 uat 的提交日志清晰）。

**辅助分支** （临时性）

- `hotfix_xxxx`: 热更新分支 （服务于 uat 与 prod）

hotfix 用于修复 uat 或 prod 出现 的 bugs，基于 uat 分支签出，bug 修复完成后并入 uat 和 prod 分支。

## 开发场景

**自己的分支 develop-yue**

创建分支：

```shell
# 基于 uat 分支签出新的开发分支
git checkout -b develop-yue uat
```

开发完成后合并:

> 注意： 先将修改提交到自己的分支

```shell
# 先切换 uat 拉取最新 code，因为 merge 的时候选取的是本地分支
git checkout uat
git pull

# rebase 变基 （基于 uat 整理好自己的新增修改）
git checkout develop-yue
git rebase uat

# 将变基好的新增内容合并到 uat
git checkout uat
git merge develop-yue

# 上传到远端
git push
```

> rebase 解决冲突: rebase 的时候，解决冲突的后的提交不是使用 commit 命令，而是执行 rebase 命令，追加参数，-- continue 选项 （继续 rebase），--abort 选项 （结束 rebase）

**hotfix 分支热更新**

创建分支：

```shell
# 基于 uat 分支签出新的热更新分支
git checkout -b hotfix_xxxx uat
```

bug 修复完成后合并到 uat:

```shell
# 先切换 uat 拉取最新 code，因为 merge 的时候选取的是本地分支
git checkout uat
git pull

# rebase 变基 （基于 uat 整理好自己的新增修改）
git checkout hotfix_xxxx
git rebase uat

# 将变基好的新增内容合并到 uat
git checkout uat
git merge hotfix_xxxx

# 上传到远端
git push
```

bug 修复完成后合并到 prod: 参照下一项 【发布的流程】

## 发布的流程

1、合并 uat 分支到 prod

```shell
# 确保本地的 uat 分支已经是最新的
git checkout prod

# 合并 uat 的修改内容
git merge --no-ff uat （简单版本: git merge uat）
```

2、打 tag

```shell
# 打 tag
git tag v1.7.21 （打标签后提交会新建一个以 tag 名命名的分支）

# 提交指定 tag 分支到远端 ：git push [remote] [tag]
git push origin v1.7.21
```

3、推送代码

```shell
# 提交 prod 分支到远端
git push
```

**补打 tag** :

```shell
# 新建一个 tag 在当前commit
git tag v1.7.21 ef9520a

# 提交指定 tag 分支到远端 ：git push [remote] [tag]
git push origin v1.7.21

# 删除本地 tag
git tag -d [tag]

# 删除远程tag
git push origin :refs/tags/[tagName]
```
