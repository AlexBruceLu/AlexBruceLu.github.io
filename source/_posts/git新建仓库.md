title: git 新建仓库，添加远程分之到本地

tags:

- git

categories:

- 工具
- git

date: 2019/12/09 18:40:43

------

@[TOC]

### 本地不存在仓库拉取远程分之



```
echo "# gin-web" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/AlexBruceLu/gin-web.git
git push -u origin master
```

### 本地已有的仓库添加到远程



```
git remote add origin https://github.com/AlexBruceLu/gin-web.git
git push -u origin master
```