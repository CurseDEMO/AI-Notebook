# Git文件三种状态
**已提交（committed）**:表示数据已经安全地保存在本地数据库中
**已修改（modified）** :表示修改了文件，但还没保存到数据库中
**已暂存（staged）**:表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中
> [!info] 说明
> 如果 Git 目录中保存着特定版本的文件，就属于 已提交 状态。 如果文件已修改并放入暂存区，就属于 已暂存 状 态。 如果自上次检出后，作了修改但还没有放到暂存区域，就是 已修改 状态。
# Git配置
## 安装
```bash
sudo apt install git-all
```
## 配置
### 外观和行为配置变量
1. **/etc/gitconfig 文件**: 包含系统上每一个用户及他们仓库的通用配置。 如果在执行 git config 时带上 --system 选项，那么它就会读写该文件中的配置变量。 （由于它是系统配置文件，因此你需要管理员或 超级用户权限来修改它。）
2. **~/.gitconfig 或 ~/.config/git/config 文件**：只针对当前用户。 你可以传递 --global 选项让 Git 读写此文件，这会对你系统上 所有 的仓库生效。 
3. **当前使用仓库的 Git 目录中的 config 文件（==即 .git/config==）**：针对该仓库。 你可以传递 --local 选 项让 Git 强制读写此文件，虽然默认情况下用的就是它。 （当然，你需要进入某个 Git 仓库中才能让该选项 生效。） 每一个级别会覆盖上一级别的配置，所以 .git/config 的配置变量会覆盖 /etc/gitconfig 中的配置变量。
### 查看配置及文件
```bash
git config --list --show-origin
```
### 用户信息
```bash
git config --global user.name "John Doe"  git config --global user.email johndoe@example.com
```
### 检查配置信息
```bash
git config --list
```
>[!tip] 原始值获取
>由于 Git 会从多个文件中读取同一配置变量的不同值，因此你可能会在其中看到意料之外的值 而不知道为什么。 此时，你可以查询 Git 中该变量的 原始 值，它会告诉你哪一个配置文件最 后设置了该值.
>```bash
>git config --show-origin key
>```


# Git基础
## 获取Git仓库

### 在已存在目录中初始化仓库
```bash
cd /home/user/my_project
git init
git add *.c
git add LICENSE
git commit -m 'initial project version'
```
### 克隆现有的仓库
```bash
git clone <url> [name]
```
## 记录每次更新到仓库
### 检查当前文件状态
```bash
git status
```
### 跟踪或暂存新文件
```bash
git add file
```
### 暂存已修改的文件
```bash
git add file
```
> [!info] git add
> 这是个多功能命令：可以用它开始跟踪新文件，或者把已跟踪的文件放到暂存区，还能用于合并时把有冲突的文件标记为已解决状态等.将这 个命令理解为“精确地将内容添加到下一次提交中”而不是“将一个文件添加到项目中”要更加合适。

### 忽略文件
创建一个名为 .gitignore 的文件，列出要忽略的文件的模式。 
#### .gitignore格式规范
• 所有==空行==或者以==#== 开头的行都会被 Git 忽略。
• 可以使用标准的 glob 模式匹配，它会递归地应用在整个工作区中。 
• 匹配模式可以以（/）开头防止递归。
• 匹配模式可以以（/）结尾指定目录。 
• 要忽略指定模式以外的文件或目录，可以在模式前加上叹号（!）取反。 

所谓的 glob 模式是指 shell 所使用的简化了的正则表达式。
星号（\*）匹配零个或多个任意字符；
[abc] 匹配 任何一个列在方括号中的字符 （这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）； 
问号（?）只 匹配一个任意字符；
如果在方括号中使用短划线分隔两个字符， 表示所有在这两个字符范围内的都可以匹配 （比如 [0-9] 表示匹配所有 0 到 9 的数字）。 
使用两个星号（\*\*）表示匹配任意中间目录，比如 a/\*\*/z 可以 匹配 a/z 、 a/b/z 或 a/b/c/z 等。

##### eg
```
# 忽略所有的 .a 文件
*.a 
# 但跟踪所有的 lib.a，即便你在前面忽略了 .a 文件 
!lib.a 
# 只忽略当前目录下的 TODO 文件，而不忽略 subdir/TODO 
/TODO 
# 忽略任何目录下名为 build 的文件夹 
build/ 
# 忽略 doc/notes.txt，但不忽略 doc/server/arch.txt 
doc/*.txt 
# 忽略 doc/ 目录及其所有子目录下的 .pdf 文件 doc/**/*.pdf
```
### 查看已暂存的更改
```bash
git diff --staged
```
### 查看未暂存的更改
```bash
git diff
```
### 提交更新
```bash
git commit
```
```bash
git commit -m 提交说明
```
```bash
git commit -a (将已跟踪的文件暂存并提交)
```
### 删除文件
```bash
git rm 
git rm -f (强制删除已暂存文件)
git rm --cached (从暂存区域移除但不从工作目录移除)
git rm log/\*.log
```
### 移动文件
```bash
git mv file_from file_to
```
## 查看提交历史
```bash
git log [option]
```
### format值

| %H  | 提交的完整哈希值                    |
| --- | --------------------------- |
| %h  | 提交的简写哈希值                    |
| %T  | 树的完整哈希值                     |
| %t  | 树的简写哈希值                     |
| %P  | 父提交的完整哈希值                   |
| %p  | 父提交的简写哈希值                   |
| %an | 作者名字                        |
| %ae | 作者的电子邮件地址                   |
| %ad | 作者修订日期（可以用 --date=选项 来定制格式） |
| %ar | 作者修订日期，按多久以前的方式显示           |
| %cn | 提交者的名字                      |
| %ce | 提交者的电子邮件地址                  |
| %cd | 提交日期                        |
| %cr | 提交日期（距今多长时间）                |
| %s  | 提交说明                        |
### git log选项

| 选项              | 说明                                                                    |
| --------------- | --------------------------------------------------------------------- |
| -p              | 按补丁格式显示每个提交引入的差异                                                      |
| \--stat         | 显示每次提交的文件修改统计信息。                                                      |
| --shortstat     | 只显示 \--stat 中最后的行数修改添加移除统计。                                           |
| --name-only     | 仅在提交信息后显示已修改的文件清单。                                                    |
| --name-status   | 显示新增、修改、删除的文件清单。                                                      |
| --abbrev-commit | 仅显示 SHA-1 校验和所有 40 个字符中的前几个字符。                                        |
| --relative-date | 使用较短的相对时间而不是完整格式显示日期（比如“2 weeks ago”）。                                |
| \--graph        | 在日志旁以 ASCII 图形显示分支与合并历史。                                              |
| \--pretty=      | 使用其他格式显示历史提交信息。可用的选项包括 oneline、short、full、fuller 和 format（用来定义自己的格式）。 |
| \--oneline      | \--pretty=oneline \--abbrev-commit 合用的简写。                             |
| --no-merges     | 隐藏合并提交                                                                |
| --decorate      | 查看各个分支当前所指的对象                                                         |
### 限制输出长度

| 选项                | 说明                    |
| ----------------- | --------------------- |
| -\<n>             | 仅显示最近的 n 条提交。         |
| --since, --after  | 仅显示指定时间之后的提交。         |
| --until, --before | 仅显示指定时间之前的提交。         |
| --author          | 仅显示作者匹配指定字符串的提交。      |
| --committer       | 仅显示提交者匹配指定字符串的提交。     |
| --grep            | 仅显示提交说明中包含指定字符串的提交。   |
| -S                | 仅显示添加或删除内容匹配指定字符串的提交。 |
## 撤销操作
```bash
git commit --amend
```
重新提交暂存文件并更改提交信息
### 取消暂存的文件
```bash
git reset HEAD file_name
```
### 撤销对文件的修改
```bash
 git checkout -- file_name

```
## 远程仓库的使用
### 查看已配置的远程仓库
```bash
git remote -v(会显示需要读写远程仓库使用的 Git 保存的简写与其对应的 URL)
```
### 添加远程仓库
```bash
git remote add  <shortname> <url>
```
### 从远程仓库抓取与拉取
```bash
git fetch <remote> 不合并
git pull <remote> 自动合并当前所在分支
```
### 推送到远程仓库
```bash
git push <remote> <branch>
```
### 查看某个远程仓库
```bash
git remote show [<remote>]
```
### 远程仓库的重命名与移除
```bash
git remote rename old_name new_name
git remote rm remote_name
```
## 打标签
```bash
git tag (列出所有标签)
git tag -l (按照通配符列出标签)
```
### 创建标签 
Git 支持两种标签：
**轻量标签**（lightweight）与**附注标签**（annotated）。
轻量标签很像一个不会改变的分支——它只是某个特定提交的引用。 而附注标签是存储在 Git 数据库中的一个完整对象， 它们是可以被校验的，其中包含打标签者的名字、电子邮件 地址、日期时间， 此外还有一个标签信息，并且可以使用 GNU Privacy Guard （GPG）签名并验证。 通常会建 议创建附注标签，这样你可以拥有以上所有信息。但是如果你只是想用一个临时的标签， 或者因为某些原因不 想要保存这些信息，那么也可以用轻量标签。
### 附注标签
```bash
git tag -a version -m "message"
```
### 轻量标签
```bash
git tag content
```
### 显示标签信息
```bash
git tag show
```
### 后期打标签
```bash
git tag 内容 hash值(部分或全部)
```
### 共享标签
```bash
git push <branch> <tagname>/--tag(全部标签)
```
### 删除标签
```bash
git tag -d <tagname>
git push <origin> --delete <tagname> (删除远程仓库的标签)
```
### 检出标签
```bash
git checkout -b <branch> <tagname>(创建新分支)
```
## Git别名
```bash
git config --global alias.缩写 完整命令名 (缩写等价于完整命令名)
git config --global alias.命令名 !外部命令名 (命令名引用外部命令名)
```
# Git分支
## 分支简介
### 概念Page70-72
![[progit.pdf#page=70]]
### 分支创建
```bash
git branch branch_name
```
### 分支切换
```bash
git checkout branch_name
```
### 分支创建并切换
```bash
git checkout -b breach_new [远程_branch] (创建一个分支并切换到该分支[且建立在远程_branch上])
```
### 一个分支合并另一个分支
```bash
git checkout branch_old
git merge branch_else
```
### 删除分支
```bash
git branch -d branch_del
```
### 合并冲突
![[progit.pdf#page=83]]

## 分支管理
### git branch
```bash
git branch(显示所有分支)
git branch -v(查看每一个分支最后一次提交) --merged(查看那些分支已合并到当前分支) --no-merged(查看那些分支未合并到当前分支) 
git branch -v(查看每一个分支最后一次提交) --merged(查看那些分支已合并到branch_name分支) --no-merged(查看那些分支未合并到branch_name分支) branch_name
```
## 分支开发工作流
![[progit.pdf#page=87]]
## 远程分支origin/branch
```bash
git ls-remote <remote>(获得远程引用完整列表)
git remote show <remote>(获取远程分支的更多信息)
git checkout -b <branch> <remote/branch>(创建跟踪分支到指定分支) == git checkout --track <remote>/<branch>
git branch -u/--set-upstream-to <remote>/<branch> 使当前分支跟踪一个远程/上游分支
git branch -vv(列出所有的本地分支及其信息)
```
## 变基
```bash
git rebase master(将当前分支移动到master分支后)
git rebase 1 2 3(将3从2与3的共同祖先分离,放到1后面去)
git rebase <target_branch> <branch>(将branch变基到target_branch后)
git rebase <remote>/<branch>(检查哪些提交是我们的分支上独有的,检查其中哪些提交不是合并操作的结果,检查哪些提交在对方覆盖更新时并没有被纳入目标分支,把查到的这些提交应用在 <remote>/<branch> 上面) == git pull --rebase
```
**不要把已经「公开」的提交拿来变基.
总的原则是，只对尚未推送或分享给别人的本地修改执行变基操作清理历史， 从不对已推送至别处的提交执行 变基操作，这样，你才能享受到两种方式带来的便利。**
# 服务器上的Git
## 协议
**本地协议（Local）
HTTP 协议
SSH（Secure Shell）协议
 Git 协议**
# GitHub
```bash
git ls-remote <remote>/<url> (显示所有引用和分支,包括假分支)
git fetch <remote> refs/pull/.../head(抓取合并请求分支的最后一个提交记录)
git merge FETCH_HEAD (合并那一个分支)
```