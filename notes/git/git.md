# Git 命令总结

## 设置全局属性
- git config --global user.name "用户名"
- git config --global user.email "邮箱"

## Git 仓库属性查看
- git status 查看当前仓库的状态
- git diff 查看修改的内容
- git log 查看所有提交日志 简易内容： --pretty=oneline
- git reflog 查看命令日志
- git checkout 文件名 撤销修改

## 版本控制
- git reset --hard HEAD^ 回退到上个版本
- git reset --hard 版本号 回退到某个版本

## 本地仓库
- git init 初始化一个仓库
- git add 文件名 将文件添加到临时仓库
- git commit -m "注释信息" 提交到本地仓库

## 远程仓库
- ssh-keygen -t rsa -C "标记信息" 创建一个 ssh 公私钥，通常 id_rsa 私钥 id_rsa.pub 公钥
- 将公钥加载到GitHub/gitee/gitlab
- ssh -T git@github.com 测试连接 GitHub
- git remote add origin 远程仓库的地址	关联远程仓库
- git remote rm origin		删除远程仓库关联
- git pull origin master		获取远程仓库内容
- git push origin  master		将本地内容上传远程仓库
- git clone 远程仓库的地址		克隆远程仓库内容
- 注意：添加文件名：.gitignore	将某些文件名写入该文件，在每次获取或上传远程仓库时，可忽略该文件（避免某些大文件每次上传下载）

## rebase 处理冲突
- git fetch origin 获取远程仓库最新代码
- git rebase origin/master 合并远程更新的分支
- git add 已解决的文件名 将解决冲突后的文件添加
- git rebase --continue 继续检查是否还有冲突（还有冲突重复上一步，没有冲突自动切回原来的分支）
- git push origin 将分支推送至远程

## 合并冲突
当两个分支或本地和远程仓库合并时，即：git merge 或 git pull 出现冲突时，对于文本文件的冲突仅需进入该文件将不需要保留的部分文件进行删除即可，对于二进制文件，可以使用 git checkout 文件名 --ours/theirs （ours保留本分支或本地仓库文件/theirs保留其他分支或远程仓库文件）进行选择型保留

## 分支管理
- git branch 分支名	创建分支（不加分支名为查看分支，确认自己目前所在分支）
- git checkout 分支名 切换分支（-b 创建分支并且换）
- git merge 分支名 合并分支（在主分支上合并，分支名为要被合并的分支）
- git push --set-upstream origin 分支名（第一次上传新分支）
- 
### 分支同步
- 本地新分支推送到远程
  - git push origin 分支名
- 远程新分支本地没有
  - git fetch 将更新取回本地
  - git branch -a 查看所有分支（包括远程）
  - git branch 查看本地分支
  - git checkout 分支 切换分支
- 本地分支删除远程也想删除
  - git branch -d 分支名 删除本地分支
  - git push origin -d 分支名 删除远程分支
- 远程分支已删除本地同步记录
  - git remote show origin 查看远程分支和本地分支的对应关系
  - git remote prune origin 删除本地中远程已删除分支

## 标签管理
- git tag 标签名 [id]	（创建标签，不加标签名为查看标签，为指定 id 打标签）
- git tag -a 标签名 -m "标签信息"	（指定标签信息）
- git checkout 标签名		（切换到指定标签）
- git show 标签名	（查看标签的说明文字）
- git tag -d 标签名	（删除标签）
- git push origin 标签名	（推送标签到远程）
- git push origin --tags	（推送全部未推送过的标签到远程）

### 删除远程标签
- 先删除本地：git tag -d 标签名
- 再删除远程：git push origin ：refs/tags/标签名
