gitignore 忽略.DS_Store 文件

1、创建 gitignore 文件，写入.DS_Store 和\*/.DS_Store

```
S1: touch .gitignore #创建 gitignore 隱藏文件
S2: vim .gitignore #编辑文件，加入指定文件
```

2、经常在其他文件夹下面也都会生成.DS_Store 文件，所以我们需要全局 ignore 该文件 创建.gitignore_global 步骤同一，然后修改它

```
S1: .vi ~/.gitignore_global
S2: 在 gitignore_global 中写入：
.DS_Store
*/.DS_Store
```

3、执行

```
git config --global core.excludesfile ~/.gitignore_global
```

4、若检查发现修改中仍然有.DS_Store 文件，则可能原因是在早期的提交中就已将.DS_Store 文件提交到 repository 了。如果一个文件已经被提交，那么就算这个文件已经被写到 gitignore 文件中，它的修改仍然会被追踪到。
所以我们需要手动将 repository 中的.DS_Store 文件移除，可以使用以下命令移除：

```
git rm --cached .DS_Store//移除当前文件夹下的.DS_Store 文件
find . -name .DS_Store -print0 | xargs -0 git rm --ignore-unmatch
//移除文件夹下的所有.DS_Store 文件
```

然后最后再检查一下会发现已经搞定了 😊
