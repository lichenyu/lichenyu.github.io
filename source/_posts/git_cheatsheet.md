title: Git Cheatsheet
date: 2018/10/19
categories:
- Program Development
tags:
- Git
---


- 初始化
`git init`
- 克隆远端repo
`git clone <remote_repo_url> <local_repo_name>`
- 添加远端仓库
`git remote add <remote_repo_name> <remote_repo_url>`
- 列出远端仓库
`git remote`
- 获取远端仓库到本地
`git fetch <remote_repo_name>`
- （本地）重命名远端仓库
`git remote rename <old_name> <new_name>`
- （本地）重设远端仓库URL
`git remote set-url <remote_repo_name> <new_url>`
- （本地）删除远端仓库
`git remote rm <remote_repo_name>`
---

- 列出本地分支
`git branch`
- 切换本地分支
`git checkout <local_branch_name>`
- 删除本地分支
`git branch -D <branch_name>`
- 创建本地分支追中远端分支
`git checkout --track <remote_repo_name/remote_branch_name>`  本地创建名为remote_branch_name的分支，追踪
`git checkout -b <local_branch_name> <remote_repo_name/remote_branch_name>`  本地创建local_branch_name的分支，追踪
- 本地分支从某远端分支来pull
`git pull <remote_repo_name> <remote_branch_name>:<local_repo_name/local_branch_name>`
- 本地分支push到远端分支
`git push <remote_repo_name> <local_branch_name>`  若local_branch_name不存在，则新建
`git push <remote_repo_name> <local_branch_name>:<remote_branch_name>`  远端as remote_branch_name
- 分支merge其他分支冲突
`git checkout <master>`
`git merge <bugfix>`
`git mergetool`
`git status`
`git commit`

---

- 检查当前修改状态  对比上次commit
`git status`
- 查看未暂存/已暂存的修改  对比上次commit
`git diff [--staged]`
- 查看提交历史
`git log`
`-数字    最近若干次`
`-p    显示差异`
`--stat    文件变化情况`

- 添加内容到下一次提交中  添加对文件（可以是目录）的跟踪；暂存已修改文件
`git add <file_name>`
- 提交
`git commit`
`-m <msg>    附带提交信息`
`-a    跳过add暂存直接commit`

- 取消已修改未缓存的文件
`git checkout -- <file_name>`
- 取消已缓存的文件  取消已经staged的文件（比如为了从分成多次提交，下次再提交）
`git reset HEAD <file_name>`
- reset到某次commit
`git reset --hard <commit_hash>`
`--soft 保留当前修改`
- 修改最后一次commit  修改commit文件或信息，使用当前的暂存区域快照更新后，提交。
`git commit --amend`
- push时文件大小超出限制
`git rm --cached <file_path/file_name>`
`.gitignore`
`git commit --amend`

---

- 修改已经提交的commit message
  1. git rebase -i HEAD~8                //8为距最后提交的倒数第8次
  2. pick -> edit
  3. git commit --amend
  4. git rebase --continue
  5. git push -f <remote> <branch>

