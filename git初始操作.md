- Git 全局设置
```
git config --global user.name "Administrator"
git config --global user.email "admin@example.com"
```

- 创建新版本库
```
git clone git@gitlab.wayne.com:root/test.git
cd test
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master
```

- 已存在的文件夹
```
cd existing_folder
git init
git remote add origin git@gitlab.wayne.com:root/test.git
git add .
git commit -m "Initial commit"
git push -u origin master
```

- 已存在的 Git 版本库
```
cd existing_repo
git remote rename origin old-origin
git remote add origin git@gitlab.wayne.com:root/test.git
git push -u origin --all
git push -u origin --tags
```
- 如何撤销commit
```
git reset --soft HEAD^

这样就成功撤销了commit，如果想要连着add也撤销的话，--soft改为--hard（删除工作空间的改动代码）。

命令详解：

HEAD^ 表示上一个版本，即上一次的commit，也可以写成HEAD~1
如果进行两次的commit，想要都撤回，可以使用HEAD~2

--soft
不删除工作空间的改动代码 ，撤销commit，不撤销git add file

--hard
删除工作空间的改动代码，撤销commit且撤销add

另外一点，如果commit注释写错了，先要改一下注释，有其他方法也能实现，如：

git commit --amend
这时候会进入vim编辑器，修改完成你要的注释后保存即可。
```