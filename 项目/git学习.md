`git status `
使用status命令确认工作树和索引的状态。

`git log`
使用log命令，我们可以在数据库的提交记录看到新的提交。

我们可以用log命令来确认数据库的历史记录是否准确。指定--graph选项，能以文本形式显示更新记录的流程图。指定--oneline选项，能在一行中显示提交的信息。
` git log --graph --oneline`

## git远程提交
```bash

git status 
git log
commit 2f7e2b68775cdbedd16589ddccbc6621b6239a9e
Author: congzesheng <congzesheng@machloop.com>
Date:   Mon Jul 11 10:25:15 2022 +0800

    采集策略ip6支持
git show 2f7e2b68775cdbedd16589ddccbc6621b6239a9e --stat

git log --oneline
git reset HEAD .
git status
```

git  新建分支
```bash
git pull origin master
git branch <分支名>
git checkout <分支名>
git push origin <分支名>
```

centos 设置ssh clone
```
ssh-keygen -t rsa -C "your_email@youremail.com"
在gitlab中 settings 中 复制刚才生成的 id_rsa.pub 公钥，Add SSH key

```