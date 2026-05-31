---
title: Git MAN
date: 2026-5-30
categories: [UNIX,CLI,MAN]
tags: [UNIX,CLI,MAN]
---



# Git Manual

[TOC]

#### 0. 背景知识 

```zsh
brew install git
```

使用Git可以方便地管理项目及上传保存笔记。为了让项目文件干净整洁，应当遵守一下原则：

1. **提交前应拉取pull/clone最新代码**：防止本地版本不同
2. **一次提交add只做一件事**：一个功能一个commit
3. **commit应保持简洁规范的格式**：如feat(feature)，refactor等
4. **不应提交临时文件**：不要把构建文件和无意义文件提交上仓库
5. **上传push前应自查**：检查status、diff等，确认上述原则已遵守

------

#### 1. 账户配置

```zsh
git config -global user.name"$NAME" # 显示名，不是登陆用户名
git config -global user.email"xxx@xxx.com" # 与Github注册邮箱一致
```

**只需要执行一次即可。**

#### 2. 添加仓库

##### 本地新建

```zsh
git init
```

在当前文件夹初始化，这会创建一个.git文件，作为仓库根目录的标志。

##### 远程仓库

```zsh
git clone https://xxx.git
```

修改远程仓库必须重新拉取，这样的仓库自带.git，不要再init，特别是在其子文件夹里。

如果不小心重新init了，把.git文件删除即可。

#### 3. 添加文件

```zsh
git add ./xxx/ # /代表递归提交其子文件
git add ./xxx.c
git status
```

一次add、commit、push只应添加一个修改，修改完毕应该先检查状态，以防提交错误。（能多个add与commit分开一次性提交，还是得封城两次push

不想提交的文件应当写入.gitignore文件。

```.gitignore
.DS_Store # 不想要跟踪的文件直接写入（全局生效）
Thumbs.db
*.log # 星号是通配符，不跟踪所有log
*.tmp
**/temp/ # 不跟踪所有层级里的temp文件夹
!keep.log # 除keep.log外
/Senshado.log # 只针对当前文件夹生效
```

有一点容易混淆的是，\**/是专门针对**文件夹**全层级递归的，而文件一般**直接写入就是全局生效**；temp/只对当前文件夹生效，而文件必须加**/**Senshado.log才能达到那种目的。

.gitignore文件也应提交。

如果提交空文件夹结构，可以在文件中加入.gitkeep：

```zsh
touch ./xxx/.gitkeep
```

添加完成要及时commit：

```zsh
git commit -m
```

| 前缀     | 含义                 |
| -------- | -------------------- |
| feat     | 新建功能与内容       |
| fix      | 修复错误与Bug        |
| docs     | 添加说明文档等       |
| style    | 添加样式与外观       |
| refactor | 代码结构优化         |
| test     | 添加测试用文件或实例 |
| chore    | 其他除上述之外的调整 |
| perf     | 性能优化             |

#### 4. 分支管理

```zsh
git branch # 查看分支
git branch -vv
git branch dev # 添加dev分支（分支名自己起吗？
git checkout dev # 切换至dev分支
git merge dev # 将dev分支与main合并
```

查看记录：

```zsh
git log
```

#### 5. 仓库管理

```zsh
git remote -v
```



```zsh
git remote set-url origin https://xxx.git # 将orgin设置成“这个仓库”
git remote add origin https://xxx.git
```



```zsh
git remote rename origin repo1
gir remote remove repo1
```



#### 6. 推送管理

```zsh
git push -u orgin main
git push -set-upstream origin main
```



```zsh
git mv old.md new.md
git mv old/ new/
```

改完得重新add与commit，再push

```zsh
git add -u
```

如果在本地保留，在远程移除

```zsh
git rm -cached file.md
```

如果之前操作错误，应当在本地先进行revert：

```zsh
git revert HEAD
git push
```

如果还没提交，可以选择撤销添加但分为保留暂存、保留或不保留本地修改：

```zsh
git reset -soft HEAD~1
```



```zsh
git reset -mixed HEAD~1
```



```zsh
git reset -hard HEAD~1
```



#### 7. GitHub Token操作

使用SSH登陆一般比Token登陆相对更安全，因其可以额外设置passphase。

同时，对于个人开发者，SSH不需要管权限问题，这可能方便一些，但对不同SSH密钥的控制力就弱了。

不过，如果使用Fine-grained-Token也有方便之处，无需使用命令行生成公钥与私钥，可以具体配置权限，但也因此必须设置GitHub pages上的权限栏，否则会报错403。

##### SSH配置方法

