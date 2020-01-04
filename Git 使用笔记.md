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