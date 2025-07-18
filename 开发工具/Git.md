# Git


|                 git命令                  |           效果            |
| --------------------------------------- | ------------------------- |
| git push -u origin HEAD:refs/for/master | 代码提交                   |
| git rm -r --cached .                    | 清除.gitignor信息,重新加载 |
| git reset                               | 清除暂存区,工作区不变       |
| git reset --soft HEAD^                  | 撤销上一次发布,工作区不变   |

> ssh-keygen -t rsa -C "SSH"
> cat ~/.ssh/id_rsa.pub
> git remote add origin git@github.com:VanGame666/仓库名.git








![](vx_images/496154534353476.png =700x)

![](vx_images/306487408049241.png =700x)

---
- 新建文件--->Untracked
- 使用add命令将新建的文件加入到暂存区--->Staged
- 使用commit命令将暂存区的文件提交到本地仓库--->Unmodified
- 如果对Unmodified状态的文件进行修改---> modified
- 如果对Unmodified状态的文件进行remove操作--->Untracked
![](vx_images/100674696148268.png =700x)

---
- Git增加分支显示, 打开 ~/.bashrc ，在结尾添加：


```
# show git branch name
function git-branch-name() {
  git symbolic-ref HEAD 2>/dev/null | cut -d"/" -f 3
}
function git-branch-prompt() {
  local branch=`git-branch-name`
  if [ $branch ]; then
    printf "(%s)" $branch
  fi
}
function get-PS1() {
  local branchname=`git-branch-prompt`
  printf "\n\e[32m%s\e[36m%s\e[35m %s \e[33m%s\e[36m %s \e[0m \n%s " "\u" "[\#]" "MSYS" "\w" $(git-branch-prompt) "\$"
}
function show-PS1() {
  if [ -d "$(pwd)/.git" ]; then
    export PS1=`get-PS1`
  else
    export PS1="\n\[\e[32m\]\u\[\e[36m\][\#]\[\e[35m\] MSYS \[\e[33m\w\]\[\e[0m\] \n\$ "
  fi
}
function cd(){
  builtin cd "$@" && show-PS1
}
show-PS1
```