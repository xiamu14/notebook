# Git 使用笔记

## 分支(branch)

1. 为分支添加描述

```bash
git branch --edit-description <branch-name>
```

2. 开启 “描述信息 push 添加到 merge 信息”

   ```bash
   git config --global merge.branchdesc true
   ```

3. 查看分支的描述信息

```bash
git config branch.<branch-name>.description
```

4. 查询全部远程分支的描述信息

```bash
#!/bin/sh
function aboutAll {
   local IFS=$'\n'
   for line in $(git branch -r); do
      if [[ $line =~ "HEAD" ]];then
         continue
      fi
      lineCopy=$(echo $line | sed s/[[:space:]]//g)
      description=$(git log remotes/$lineCopy --grep=about: --pretty=format:"[%cr] %s - %an")
      if [ -n "$description" ]; then
      echo "\033[36m[$lineCopy] \033[0m\n$description"
      fi
   done
}

aboutAll
```

5. 添加查看分支描述信息快捷命令

```bash
git config --global --add alias.about '!describe() { git config branch."$(git symbolic-ref --short -q HEAD)".description; }; describe'
```

6. 删除远程分支和本地分支

```bash
git branch -r # 查看远程分支
git branch -r -d origin/<branch-name>
git push origin :<branch-name>
git branch -d <branch-name>
```

7. 使用 commit 添加分支描述并查看

   ```bash
   git commit --allow-empty  # 添加空白的commit[即没有修改时添加一条 commit 用于描述分支]
   git log <branch-name> --grep=<keywords> # 通过关键词查询 commit 信息，分支描述通常以 “about:”, "desc:" 等词开头 
   ```


## git commit template

目前在用的 git commit template 保存在 github gist 里面,由于维护时间较短，所以采用比较简易的 template 。

业界完善的 template 的有 Angular 提供的 template，可以直接参考[这篇文章](https://segmentfault.com/a/1190000009048911)，比较系统的描述了 Angular git commit template 的内容。

## git如何从仓库中删除已经被跟踪的文件

- 首先在 .gitignore 文件中需要忽略的文件/文件夹
- 删除本地文件、文件夹
- 删除 git 库里的文件

```bash
git rm --cached xxx [-r]
```