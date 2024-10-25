# Git 知识点

## 全局配置文件路径

```
C:\Users\<User>\.gitconfig
```

## 设置配置
```shell
git config  user.name "louisgeek"
git config  user.email "louisgeek@qq.com"
//全局
git config --global user.name "louisgeek"
git config --global user.email "louisgeek@qq.com"
//
git config --global http.sslVerify "false"
```

## 查询配置
```shell
git config --list
//全局
git config --global --list
```

## 取消配置
```shell
git config --global --unset http.sslbackend
```



```shell
git add .
```

```shell
git commit -m "修改文件"
```

```shell
git push -u origin main
```






## 修改 .gitignore 文件后立即生效

```shell
#清除缓存
git rm -r --cached .
#重新 trace file
git add .
#提交
git commit -m "修改 .gitignore 文件"
#可选，按需同步到 remote
git push origin main
```

## 修改提交信息
```shell
git commit --amend --author "louisgeek <louisgeek@qq.com>"
```

## 删除仓库所有提交的历史记录，成为一个干净的新仓库

```shell
1.Checkout

   git checkout --orphan latest_branch

2. Add all the files

   git add -A

3. Commit the changes

   git commit -am "commit message"


4. Delete the branch

   git branch -D main

5.Rename the current branch to main

   git branch -m main

6.Finally, force update your repository

   git push -f origin main

```











