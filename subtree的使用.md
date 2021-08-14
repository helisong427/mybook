# git subtree 的使用

git subtree的主要命令有：

```
git subtree add   --prefix=<prefix> <commit>
git subtree add   --prefix=<prefix> <repository> <ref>
git subtree pull  --prefix=<prefix> <repository> <ref>
git subtree push  --prefix=<prefix> <repository> <ref>
git subtree merge --prefix=<prefix> <commit>
git subtree split --prefix=<prefix> [OPTIONS] [<commit>]
```

## 准备

我们先准备一个仓库叫photoshop，一个仓库叫libpng，然后我们希望把libpng作为photoshop的子仓库。
photoshop的路径为`https://github.com/test/photoshop.git`，仓库里的文件有：

```
photoshop
    |
    |-- photoshop.c
    |-- photoshop.h
    |-- main.c
    \-- README.md
```

libPNG的路径为`https://github.com/test/libpng.git`，仓库里的文件有：

```
libpng
    |
    |-- libpng.c
    |-- libpng.h
    \-- README.md
```

以下操作均位于父仓库的根目录中。

## 在父仓库中新增子仓库

我们执行以下命令把libpng添加到photoshop中：

```
git subtree add --prefix=sub/libpng https://github.com/test/libpng.git master --squash
```

(`--squash`参数表示不拉取历史信息，而只生成一条commit信息。)

执行`git status`可以看到提示新增两条commit：
![image](https://image-static.segmentfault.com/328/702/3287022680-5a0a9edfaf6d9_articlex)

`git log`查看详细修改：
![image](https://image-static.segmentfault.com/181/872/1818720196-5a0a9edfbd94d_articlex)

执行`git push`把修改推送到远端photoshop仓库，现在本地仓库与远端仓库的目录结构为：

```
photoshop
    |
    |-- sub/
    |   |
    |   \--libpng/
    |       |
    |       |-- libpng.c
    |       |-- libpng.h
    |       \-- README.md
    |
    |-- photoshop.c
    |-- photoshop.h
    |-- main.c
    \-- README.md
```

注意，现在的photoshop仓库对于其他项目人员来说，可以不需要知道libpng是一个子仓库。什么意思呢？
当你`git clone`或者`git pull`的时候，你拉取到的是整个photoshop(包括libpng在内，libpng就相当于photoshop里的一个普通目录)；当你修改了libpng里的内容后执行`git push`，你将会把修改push到photoshop上。
也就是说photoshop仓库下的libpng与其他文件无异。

## 从源仓库拉取更新

如果源libpng仓库更新了，photoshop里的libpng如何拉取更新？使用`git subtree pull`，例如：

```
git subtree pull --prefix=sub/libpng https://github.com/test/libpng.git master --squash
```

## 推送修改到源仓库

如果在photoshop仓库里修改了libpng，然后想把这个修改推送到源libpng仓库呢？使用`git subtree push`，例如：

```
git subtree push --prefix=sub/libpng https://github.com/test/libpng.git master
```

## 简化git subtree命令

我们已经知道了git subtree 的命令的基本用法，但是上述几个命令还是显得有点复杂，特别是子仓库的源仓库地址，特别不方便记忆。
这里我们把子仓库的地址作为一个remote，方便记忆：

```
git remote add -f libpng https://github.com/test/libpng.git
```

然后可以这样来使用git subtree命令：

```
git subtree add --prefix=sub/libpng libpng master --squash
git subtree pull --prefix=sub/libpng libpng master --squash
git subtree push --prefix=sub/libpng libpng master
```



> 原文：https://baijiahao.baidu.com/s?id=1666708486591555944&wfr=spider&for=pc

