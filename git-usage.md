# 记录一些关于 git 的使用

配置可拉取分支
-------------------------------------------------------------------------------
git 在一些情况下会默认只拉取部分、单个分支, 或者你想使他仅拉取部分分支

该配置位于 `.git/config` 的 `[remote]` 小节中的 `fetch` 属性

```bash
git remote set-branches origin \*        # 改为拉取所有分支
git remote set-branches origin --add dev # 额外拉取 dev 分支
```


慎用 checkout
-------------------------------------------------------------------------------
`git checkout` 是一个较老的命令, 混合了许多功能, 使用时一不小心可能造成未预期的结果

可以使用具有其子功能的更具体的命令, 具有类似的选项

- `git switch`: 操作分支与头指针
- `git restore`: 操作本地与暂存区


方便的交互
-------------------------------------------------------------------------------
git 的许多命令都有特殊的交互模式, 通常用 `-i` 参数开启, 例如:

- `git rebase -i`: 编辑器中, 交互的任意重排序、修补、删除、挤压、编辑任意变基中的 commit
- `git add -p`: 交互拣选、切分本地差异添加到暂存区
- `git add -i`: 多功能交互添加 (可能有点复杂)
- `git add -e`: 手动编辑本地差异补丁并添加补丁到暂存区
- `git clean -i`: 交互选择清除工作区中未跟踪文件


合并工具 (mergetool)
-------------------------------------------------------------------------------
git 可以使用外部编辑器来更方便的解决合并冲突, 例如使用 vim 进行多窗口比较合并

可以使用的工具使用 `git mergetool --tool-help` 来列出

例如可以使用 `vimdiff`, `vimdiff2`

类似的还有 `difftool`


浅克隆 (shallow)
-------------------------------------------------------------------------------
`git clone` 大仓库时, 经常由于仓库过大, 导致克隆时间过长

指定 `--depth=10` 将只克隆十个 commit, 这样不用拉取过于久远的历史 (fetch 也可用该参数)

如果使用过程中需要更深的历史, 可以使用 `git fetch --depth=30`


部分克隆 (部分拉取)
-------------------------------------------------------------------------------
`git clone` 时, 可能只需要少部分 commit 的数据, 但是希望具有完整的提交历史

指定 `--filter=blob:none`, 可以仅获取一些基本的目录结构, 不拉取具体文件内容,
避免历史中大文件更改过多导致克隆时间过长

当需要使用到未拉取的历史 commit 数据时, 才会现场拉取具体文件内容,
例如 `git show` 某个 commit, 就需要拉取该 commit 及其父 commit

> [!TIP]
> 配合稀疏检出, 可以当前 commit 都能部分拉取


稀疏检出 (sparse-checkout)
-------------------------------------------------------------------------------
稀疏检出可以让你在工作区中仅加载少量文件, 配合部分克隆可以仅克隆更少的数据进行开发

稀疏检出由核心命令 `git sparse-checkout` 的子命令来控制, 常用的如下:

- `git sparse-checkout init`: 启用稀疏检出
- `git sparse-checkout disable`: 禁用稀疏检出
- `git sparse-checkout set`: 设置稀疏检出中保留的目录
- `git sparse-checkout add`: 添加稀疏检出中保留的目录
- `git sparse-checkout list`: 列出稀疏检出中保留的目录

> [!IMPORTANT]
> 在克隆时, 使用 `git clone --sparse --filter=blob:none --depth=8`
> 进行 稀疏克隆+部分克隆+浅克隆, 以极少的拉取量进行工作

> [!NOTE]
> 稀疏检出无法仅检出单个文件, 只能检出完整的目录及其子目录、子文件


手动应用 commit
-------------------------------------------------------------------------------
`git cherry-pick` 将任意的 commit 提交至当前的 HEAD 上, 非常灵活


配置 git
-------------------------------------------------------------------------------
配置一些 git 设置, 使其更方便使用, 配置文件位于 `~/.gitconfig`

### 一些实用的 git 别名

```gitconfig
[alias]
    cp = cherry-pick
    dt = difftool
    mt = mergetool
    rh = reset --hard
    sc = sparse-checkout
    sparse-clone = clone --depth=1 --filter=blob:none --sparse
    detach = switch --detach
    log-graph = -c log.showRoot=false log --graph --all
```

> [!NOTE]
> 关于为什么不用 shell 的别名, 因为我使用 bash, 而经过了 bash 别名补全就很麻烦了,
> 而 git 的别名通常能识别补全

### 配置 core

```gitconfig
[core]
	pager = less -FRXS      # less 查看时处理过长的行, 防止其破坏预览
	quotePath = false       # 中文路径等情况必备
```
