---
typora-root-url: images
typora-copy-images-to: images
---

# Git
### Git简介

<b>Git</b> 是一种分布式版本控制系统，它可以不受网络连接的限制。

### 准备阶段

进入 Git 官网下载安装包，安装完成后，打开命令行工具，进入工作文件夹。<img src="/image-20210628112647736.png" alt="image-20210628112647736" style="zoom:80%;" />

跳转到 E:\cq\git ，创建一个新的 demo 文件夹。

进入 GitHub 官网，登录账号，进入项目，点击 Code --> clone --> HTTPS，复制项目地址 https://github.com/lietome12/hello-world.git 。

### 常用操作

Git 命令列表

[git clone] [git config] [git branch] [git checkout] [git status] [git add] [git commit] [git push] [git pull] [git log] [git tag]

#### git clone

<!--从 git 服务器拉取代码-->

```
git clone https://github.com/lietome12/hello-world.git
```

#### git config

<!--配置开发者用户名和邮箱-->

```
git config user.name lietome22
git config user.email cqlietome@163.com
```

每次代码提交时都会生成一条提交记录，其中包含当前配置的用户名和密码。

#### git branch

<!--创建、重命名、查看、删除项目分支，通过 git 做项目开发时，一般都是在开发分支中进行，开发完成后合并到主干。-->

创建一个名为 `daily/0.0.0` 的日常开发分支，分支名只要<b>不包括特殊字符</b>即可

```
git branch daily/0.0.0
```

如果觉得之前的分支名称不合适，可以重命名分支 `daily/0.0.1`

```
git branch -m daily/0.0.0 daily/0.0.1
```

不带参数的 `branch` 命令可以查看当前项目分支列表

```
git branch
```

如果分支已完成使命，可通过 `-d` 参数将分支删除

```
git branch -m daily/0.0.1
```

#### git checkout

<!--切换分支-->

切换到 `daily/0.0.1` 分支，后续操作都是在该分支上进行

```
git checkout daily/0.0.1
```

#### git status

<!--查看文件变更状态-->

修改 readme.md 文件，保存。通过 `git status` 命令可以看到当前文件状态。`Changes not staged for commit`(改动的文件未提交到暂存区)

```
git status
```

![image-20210628120111773](/image-20210628120111773.png)

#### git add

<!--添加文件更改到暂存区-->

指定文件名 README.md 将该文件添加到暂存区，如果想添加所有文件，使用 `git add .` 命令即可。此时通过 `git status` 命令查看当前文件状态 `Changes to be committed`（文件已提交到暂存区）

```
git add README.md
```

![image-20210628120219430](/image-20210628120219430.png)

#### git commit

<!--提交文件变动到版本库-->

通过 `-m` 参数可以直接在命令行里输入提交描述文字

```
git commit -m '提交原因'
```

#### git push

<!--将本地代码改动推送到服务器-->

`origin` 指代的是当前的 git 服务器地址，这行命令的意思是把 `daily/0.0.1` 分支推送到服务器。

```
git push origin daily/0.0.1
```

#### git pull

<!--将服务器上的最新代码拉取到本地-->

如果其它项目成员对项目做了改动并推送到服务器，我们需要将最新的改动更新到本地。

```
git pull origin daily/0.0.1
```

如果本地的代码也有变动，拉取的代码有可能会和本地代码改动冲突。一般情况下 `git` 会自动处理这种冲突合并，但如果改动的是同一行，那就需要手动来合并代码，编辑文件，保存最新的改动。再通过 `git add .` 和 `git commit -m 'xxx'` 来提交合并。

#### git log

<!--查看版本提交记录-->

可以查看整个项目的版本提交记录，它里面包含了 `提交人`、`日期`、`提交原因`等信息

```
git log
```

提交记录可能会非常多，按 `J` 键往下翻，按 `K` 键往上翻，按 `Q` 键退出查看

#### git tag

<!--为项目标记里程牌-->

当我们完成某个功能需求准备发布上线时，应该将此次完整代码做个标记，并将这个标记好的版本发布到线上，这里以 `publish/0.0.1` 作为标记名发布。

```
git tag publish/0.0.1
git push origin publish/0.0.1
```

#### .gitignore

<!--设置哪些内容不需要推送到服务器，这是一个配置文件-->

`.gitinore` 不是 `git` 命令，而在项目中的一个文件，通过设置 `.gitignore` 的内容告诉 `git` 哪些文件应该被忽略，不需要推送到服务器。通过 `touch` 创建一个 `.gitignore` 文件。

```
touch .gitignore
```

用文本编辑器打开文件，每一行代表一个要忽略的文件或目录，如：忽略 `demo.html` 文件和 `build/` 目录。

```
demo.html
build/
```

文章地址： https://mp.weixin.qq.com/s/Q_O0ey4C9tryPZaZeJocbA

