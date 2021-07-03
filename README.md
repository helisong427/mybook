# mybook





### mybook使用

1. 编译gitbook，执行命令：

   ```
   gitbook build 或者 gitbook serve
   ```

2. 推送修改到main分支。

3. 推送_book目录到子分支mybook.io，gitbook仓库在mybook.io分支里面，mybook.io是main仓库的子仓库对应_book目录，使用git subtree命令推送（参考 gitsubtree 使用）：

   ```bash
   git subtree push --prefix=_book origin mybook.io
   ```

   

![image-20210703165327634](.\image-20210703165327634.png)
