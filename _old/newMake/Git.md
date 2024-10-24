



修改 .gitignore 文件后立即生效

```shell
#清除缓存
git rm -r --cached .
#重新 trace file
git add .
#提交
git commit -m "修改 .gitignore 文件"
#可选，按需同步到 remote
git push origin master
```

## git仓库删除所有提交历史记录，成为一个干净的新仓库

```
1.Checkout

   git checkout --orphan latest_branch

2. Add all the files

   git add -A

3. Commit the changes

   git commit -am "commit message"


4. Delete the branch

   git branch -D master

5.Rename the current branch to master

   git branch -m master

6.Finally, force update your repository

   git push -f origin master

```

```
//修改提交信息
git commit --amend --author "louisgeek <louisgeek@qq.com>"


```



```

git config  user.name "louisgeek"
git config  user.email "louisgeek@qq.com"


git config --global user.name "louisgeek"
git config --global user.email "louisgeek@qq.com"
```



```
git config  --list
git config --global --list
git config --global --unset http.sslbackend
```

