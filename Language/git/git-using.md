git 加速下载 https://ghproxy.com/

```bash
git clone https://ghproxy.com/httpsurl
```



clone 仓库，完成代码后 push 到自己的仓库。error: remote origin already exists.

```bash
// 1.删除关联的 origin 的远程库
git remote rm origin
// 2.关联自己的仓库
git remote add origin https://github.com/xxxxxx.git
// 3.推送到自己的仓库
git push origin master

// git none-fast-forward 
// https://blog.csdn.net/weixin_41287260/article/details/89742151
git push -f origin master
```



SSL_ERROR_SYSCALL，代理设置问题

```bash
git config --global --unset http.https://github.com.proxy

git config --global --unset https.https://github.com.proxy
```



git 配置 [Git ssh 配置及使用](https://blog.csdn.net/gdutxiaoxu/article/details/53573399)



git 清除历史提交记录 [Git清除历史提交记录](https://blog.csdn.net/Liu_Wd/article/details/120910899)

```bash
// 1.checkout
# orphan 参数用于创建没有 commit 记录的分支
git checkout --orphan temp_branch
// 2.add all files
git add -A
// 3.commit the changes
git commit -am "commit description"
// 4.delete the branch
git branch -D master
// 5.rename the current branch to master
git branch -m master
// 6.force update your repository
git push -f origin master

// 步骤：步骤就是把master分支复制，删除原有分支，用新的分支覆盖旧分支。从而完成分支替换，清除历史记录。注意：历史记录清除后无法回滚。
```



本地合并时遇到refusing to merge unrelated histories，原因是两个仓库不同而导致的

```bash
git pull origin master --allow-unrelated-histories
```

