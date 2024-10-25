# Git 知识点

```shell
git config --global user.name "louisgeek"
git config --global user.email "louisgeek@qq.com"
git config --global http.sslVerify "false"
```


```shell
git config --global http.sslVerify "false"
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



