# private/ — 私有增量

本目录**只存在于 `private/*` 分支**，主库任何分支都没有它。

## 模型

一个本地仓库，两个远程，按分支前缀单向路由：

```
  origin (公开)                          私库 (private)
  封板代码                                只收 private/* 分支
       │
       │  ①  需求单向同步
       │     git merge origin/main
       ▼
  private/<topic>  =  origin/main  +  private/ 下的纯增量  ──②──►  私库
```

## 两条规则

### 规则 1 · 需求从主库单向同步

私有分支**定期 `git merge origin/main`**，反向永不发生。
主库是唯一的代码权威；私有分支只是它的一个带注解的快照。

```bash
git switch private/defence-notes
git fetch origin
git merge origin/main
```

因为增量全在 `private/` 下、而主库没有这个路径，**这个 merge 永远不会冲突**。
这不是运气，是规则 2 挣来的。

### 规则 2 · 非必要不改动主库文件，只做增量

私有分支相对 `origin/main` 的 diff **应当只有 `private/` 下的新增**。

自查：

```bash
git diff origin/main...private/defence-notes --name-only | grep -v '^private/'
```

输出为空 = 合规。`pre-push` 会在推送时跑同一条检查，**不合规只警告不拦截**——
这是卫生规则不是安全规则，真有必要改主库文件时不该被它挡住。

真正拦截的只有泄漏方向，见下。

## 为什么增量收在单一目录下

三件事一起解决：

1. **merge 永不冲突**（规则 1 白拿）
2. **规则 2 变成可机械检查的**，不靠自觉
3. **泄漏可检测**：任何要推向公开远程的提交，树里出现 `private/` 就是事故，
   `pre-push` 直接拒绝

## push 拦截（`.git/hooks/pre-push`）

远程是否为私库，看标记，**fail-closed**——没打标记一律当公开：

```bash
git config remote.<name>.uoipPrivate true
```

| 检查 | 触发条件 | 处置 |
|---|---|---|
| 分支前缀 | `private/*` → 公开远程 | 🔴 拒绝 |
| 分支前缀 | 非 `private/*` → 私库 | 🔴 拒绝 |
| **内容** | **推向公开远程的 tip 树里有 `private/`** | 🔴 **拒绝** |
| 规则 2 | 私有分支改了 `private/` 以外的文件 | ⚠️ 警告，放行 |

第三条是最后一道防线，它管的是**前两条管不到的情况**：
人在 `main` 上、`private/` 目录还留着、一个 `git add -A` 就提交进去了。
前两条只看分支名，对这种情况是瞎的。

⚠️ `git push --all origin` 会**整体失败**（批次里有私有分支）。
fail-closed，是对的，但行为跟以前不同——日常用 `git push origin <branch>`。

⚠️ **不要 `git push private --mirror`**：mirror 推全部 ref
（公开分支一起带过去），而且语义包含**删除**远端多余 ref。逐个分支推。

🔴 **hook 在 `.git/` 下，不随 clone 走，换机器必须重装：**

```bash
git show private/defence-notes:private/hooks/pre-push > .git/hooks/pre-push
chmod +x .git/hooks/pre-push
```

## 配私库（建好仓库后跑一次）

```bash
git remote add private git@github.com:<you>/<private-repo>.git
git config remote.private.uoipPrivate true      # ← 漏了这行 hook 会拒推
git push -u private private/defence-notes
```

## 新建下一个私有分支

从主库切，不从别的私有分支切——每个私有分支都是 `origin/main` 的独立增量：

```bash
git fetch origin
git switch -c private/<topic> origin/main
mkdir -p private && $EDITOR private/<topic>.md
```

建议用 worktree，物理隔离，主工作区的 `git add .` 碰不到：

```bash
git worktree add ../uoip-private private/defence-notes
```

## 与 CLAUDE.md 的关系

CLAUDE.md「本仓库对公，私有笔记对私，依赖方向是单向的」——
本目录是那条规则的一个**落地形态**：引用可以从这里指向仓库里的
`file:line`，主库里不得出现本目录任何文件的路径、标题或链接。

另有一条更早的边界：演讲 outline、会议纪要、对外投稿这类材料属于
ToucanShelf，不属于 `docs/`。本目录承接的是**介于两者之间**的东西——
跟代码强绑定（满页 `file:line`）、但不宜公开的个人研究理解。

## 内容

- [`m1-defence.md`](m1-defence.md) — M1 统计答辩手册
- [`20260919-four-questions.md`](20260919-four-questions.md) — 现场四问的技术侧复盘
  （事件纪实本身在 ToucanShelf，两边刻意不重复）
- `hooks/pre-push` — 拦截器副本，换机器时取回
