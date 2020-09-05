## git常用操作

查看git版本：

```shell
git --version
```

git配置用户和邮箱：

```shell

git config --global user.name xxx
git config --global user.email xxx
git config --list 
// list默认输出
// diff.astextplain.textconv=astextplain
// filter.lfs.clean=git-lfs clean -- %f
// filter.lfs.smudge=git-lfs smudge -- %f
// filter.lfs.process=git-lfs filter-process
// filter.lfs.required=true
// http.sslbackend=openssl
// http.sslcainfo=d:/Program Files/Git/mingw64/ssl/certs/ca-bundle.crt
// core.autocrlf=true
// core.fscache=true
// core.symlinks=false
// pull.rebase=false
// credential.helper=manager
// core.repositoryformatversion=0
// core.filemode=false
// core.bare=false
// core.logallrefupdates=true
// core.symlinks=false
// core.ignorecase=true
```

提交步骤：

```shell
git init
git status
git add
git add .
git commit -m xxx
git log
```

撤销：

```shell
git checkout xxx  // 用暂存区的文件覆盖工作目录的文件
git rm --cached xxx // 删除到暂存区的文件
git reset --hard commitID // 将git仓库中指定的更新记录恢复过来，并且覆盖暂存区和工作目录
```

分支：

```shell
git branch // 查看分支
git branch xxx // 创建分支
git checkout xxx // 切换分支
git merge srcBranch dstBranch // 合并分支
git branch -d xxx // 删除分支
git branch -D xxx // 强制删除
```

暂存：

```shell
git stash // 存储临时改动
git stash pop // 恢复改动
```

远程操作：

```shell
# 初始化远程仓库，并将本地仓库上传到github
git remote add origin https://github.com/grcwunan/MyGitHub.git // 添加别名为origin
git branch -M master
git push -u origin master

git push https://xxxx master
git remote add xxx https://xxx
git push -u origin master
```

克隆：

```shell
git clone https://xxx
```

拉取：

```shell
git pull origin master // origin为远程仓库的别名，master为分支
```

其他：

```shell
ssh-keygen // 生成SSH秘钥
// 忽略文件，新建.gitignore，在其中添加忽略的文件名
```

