---
title: Git 使用笔记
tags: [Import-8bdb]
created: '2020-01-04T03:02:59.902Z'
modified: '2020-01-04T16:19:34.685Z'
---

# Git 使用笔记

## 分支(branch)

1. 为分支添加描述

```bash
git branch --edit-description <branch-name>
```

2. 查看分支的描述信息

```bash
git config branch.<branch-name>.description
```

3. 查询全部分支的描述信息

```bash
for line in $(git branch -r); do
   description=$(git config branch.$line.description)
   if [ -n "$description" ]; then
    echo "$line\n$description\n"
   fi
done
```

4. 删除远程分支

```bash
git branch -r # 查看远程分支
git branch -r -d origin/<branch-name>
git push origin :<branch-name>
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
