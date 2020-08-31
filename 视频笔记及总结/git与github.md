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

