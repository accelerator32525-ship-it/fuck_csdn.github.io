---
title: Mac MAN
date: 2026-5-30
categories: [MAC,UNIX,MAN]
tags: [UNIX, MAN]
---



# Mac Manual

[TOC]

#### 0. zsh常用命令

##### Directory

正则表达式：

1. ~ 用户主目录，即/Users/$USERNAME$（不带任何参数等效回主目录）
2. .. 返回上一级
3. . 当前目录下
4. / 根目录
5. （无斜杠） 当前目录下，与./相同

```zsh
echo $PATH | tr ':' '\n'
```

> Note
>
> 1. ~/.zprofile内的规则会在每一次启动Terminal时执行
> 2. ~/.zshrc内的规则会在每一次启动新Terminal实例时执行

```zsh
ls
```

> Note
>
> 1. -l long 以长形式显示文件
> 2. -a all 显示所有文件
> 3. -h human 以易读方式显示文件

```zsh
cd
```

```zsh
pwd # print working directory
```

> Note
>
> 显示当前工作目录

```zsh
mkdir # make directory
```

> Note
>
> ```zsh
> mkdir -p ~/Movies/Senshado
> ```
>
> 正则表达式，参看cd

```zsh
rm # remove
```

> Note
>
> 1. -r recursive 递归
> 2. -f force 强制

```zsh
cp # copy
```

> Note
>
> 1. -r recurisve 递归复制文件夹 -R 保留软链接 -a 保留所有权限

```zsh
mv # move
```

##### Files&Search

```zsh
cat # Concatenate
```

> 输出文本内容如cat "senshado.txt"

```zsh
grep
```

> 搜索文字
>
> ```zsh
> cat "War thunder.txt" | grep "BVVD's mother" || echo "none"
> ```
>
> ```zsh
> > none
> ```

##### System&Analysis

```zsh
df # disk free
```

```zsh
du # disk usage
```

```zsh
free # ram
```

##### Instructions&Procedures

```zsh
; 
```

```zsh
&&
```

```zsh
||
```

```zsh
&
```

```zsh
|
```

```zsh
> or >>
```

```zsh
$() or $VAR
```

```zsh
'' or ""
```

> Note
>
> 1. ；sequential 顺序执行，不管前面是否执行成功
>
> 2. && and 与执行，前面执行成功才执行后面
>
> 3. || or 或执行
>
> 4. & 后台执行，不影响当前的终端，常用于脚本并行
>
> 5. | tunnel 管道，传输前一个命令的结果
>
> 6. \> 输出命令结果并覆盖至文件；>> 输出命令结果添加至文件末尾，常用与echo连用：
>
>    ```zsh
>    echo "Hello" > ~/hello.txt
>    ```
>
> 7. $() 执行顺序，先执行括号内的命令，用于替代命令操作数的内容; $VAR 取内存里的数据
>
>    ```zsh
>    echo '$(ls)'
>    sensha="Type 89 I-Goo"
>    echo '$sensha'
>    ```
>
> 8. 'string' 内不解析任何特殊字符；"string"解析所有特殊字符

#### 其他

```zsh
source
```

> 执行脚本，和直接运行的区别是，source执行后当前终端会继承脚本对环境的修改，如脚本：
>
> ```sh
> cd /tmp
> ```
>
> 直接执行不会有环境修改

```zsh
eval
```

> 将后面的文本当作命令执行，比如
>
> ```zsh
> eval 'echo "hello"'
> ```
>
> 输出hello
>
> 有时和$()搭配使用







#### 1. Homebrew安装

```zsh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

#### 2. 权限配置

```zsh
nano ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

> Note
>
> 1. eval：将后面的文字当作命令执行，比如下面brew shellenv输出的文字
> 2. brew shellenv：一个命令，执行后输出设定环境变量的一堆命令，但是是以文字形式输出的，于是需要eval解析

或者：

```zsh
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
```

#### 3. 安装git

```zsh
brew install git
```

brew会自动处理$PATH，通过软连接至/opt/homebrew/bin/下

或者：

```zsh
git --version
```

如未安装，会自动安装

#### 3. 安装Python

```zsh
brew install python
```

将会安装至/opt/homebrew/bin/python3，安装的是py3.xx，xx是发行年份数字。

> 可选：设置alias
>
> ```zsh
> echo 'alias py="python3"' >> ~/.zshrc
> echo 'alias pip="python3 -m pip"' >> ~/.zshrc
> ```
>
> 这里存到zshrc中，是因为**alias没有父继承特性**。
>
> 同样，也可以nano打开~/.zshrc，手动写入

##### 1.  配置Python虚拟环境

```zsh
python3 -m venv
source venv/bin/activite
```

> source：执行后面的脚本，这里执行的是activite脚本。

执行完毕后，**$USERNAME$@admindeMacBook-Pro ~ %**前面会出现(venv)

```zsh
pip install ... # 注意必须先设置alias
```

#### 4. 其他实用软件

```zsh
brew install ...
```

1. FFmprg
2. android-platform-tools
3. yt-dlp

#### 5. 隐私文件/卸载垃圾管理

**谨慎操作文件：**(起始路径在主文件夹～/)

```zsh
cd Library/Preferences
cd Library/Application\ Support
cd Library/Caches # 系统会自动清理
cd Library/Containers/
```

**垃圾文件夹：**(起始路径在主文件夹～/)

```zsh
rm -rf Library/Logs/
rm -rf /private/var/log/ && rm -rf /private/var/tmp
rm -rf Library/Safari/Downloads/
```

清理除com.apple.开头的文件之外的即可，以Preferences里的文件举例：

| 名称                                                         | 作用                        | 建议              |
| :----------------------------------------------------------- | --------------------------- | ----------------- |
| diagnostics_agent.plist                                      | 故障诊断上传                | ❌删除可能影响功能 |
| familycircled.plist                                          | 家人共享                    | ❌删除可能影响功能 |
| group.com.apple.photolibraryd.private.plist                  | iCloud 照片同步             | ❌删除没有意义     |
| loginwindow.plist                                            | 控制登陆界面的行为          | ❌删除可能影响功能 |
| mbuseragent.plist                                            | Media Player等相关设置      | ❌删除可能影响功能 |
| MobileMeAccounts.plist                                       | Apple Account相关缓存及痕迹 | ✅删除防止隐私泄露 |
| no_backup                                                    | 标记文件，防止时间机器备份  | ❌删除可能影响功能 |
| org.videolan.vlc                                             | VLC播放器相关               | ✅无所谓           |
| org.videolan.vlc.plist                                       | 同上                        | ✅无所谓           |
| pbs.plist                                                    | 剪贴板相关                  | ❌删除可能影响功能 |
| sharedfilelistd.plist                                        | 管理最近使用联想等          | ❌删除可能影响功能 |
| systemgroup.com.apple.icloud.searchpartyd.sharedsettings.plist | 位置共享相关                | ⚠️删除需要重新配置 |
| TokenBucketRateLimiter.plist                                 | 系统网管调度等等            | ❌删除可能影响功能 |
| ContextStoreAgent.plist                                      | 系统上下文建议代理          | ❌删除可能影响功能 |



