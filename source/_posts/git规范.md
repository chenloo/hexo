---
title: git规范
author: 陈龙
tags:
  - git
  - 学习
categories:
  - git
keywords:
  - git
  - 规范
date: 2023-06-08 08:28
---
## 分支管理

1.  master主分支：master主分支始终保持稳定的可发布版本 说明：只有项目组主程才拥有master主分支的管理权限（例如其他分支合并到master必须由主程操作）
2.  dev 开发分支：dev开发分支为不稳定版本，可能存在功能缺失，但已有的功能必须是完整的 说明：原则上不允许直接在dev分支上进行功能开发，必须新建feature分支进行开发
3.  hotfix-[问题名称 | bug编号] 紧急热修复分支：从master分支创建，横线后面跟上问题名称或者对应的bug编号，仅仅适用于生产线问题紧急修复！ 说明：修复完成，测试通过，合并到master和dev分支上，然后将此分支删除
4.  feature-[功能名称] 功能开发分支：从dev分支创建，横线后跟功能名称，用于新功能开发，每天下班前push提交到远程 说明：开发完成以后，在远程发起向dev分支的合并请求，由指定的CodeReview人员审查通过以后进行合并，并删除该分支
5.  bugfix-[bug编号] 问题修复分支：从dev分支创建，用于修改测试提出的bug，横线后跟bug编号 说明：修复以后，在远程发起向dev分支的合并请求，并指定提交者自身（或其他人）作为CodeReview，合并以后删除该分支
6.  refactor-[重构名称] 重构分支：从dev分支创建，用于代码的重大规模重构（小规模重构创建feature分支即可）
7.  说明：重构以后，必须经过严格测试通过，才能向dev分支合并。

## Commit提交规约

### 提交信息格式

```
<type>(<scope>): <subject> #header
// 空一行
<body>
// 空一行
<footer> 
```

说明：

-   Header部分只有一行，包括三个字段：type（必需）、scope（可选）和subject（必需）。body 和 footer可省略
-   type用户说明本次commit类型，只允许使用下面7个标识：

1.  **feat**：新功能（feature）
2.  **fix**：修补bug
3.  **docs**：文档（documentation）
4.  **style**： 格式（不影响代码运行的变动）
5.  **refactor**：重构（即不是新增功能，也不是修改bug的代码变动）
6.  **test**：增加测试
7.  **chore**：构建过程或辅助工具的变动

-   scope：用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。
-   subject：是 commit 目的的简短描述，不超过50个字符。

### 列表项commit提交频率

-   每天下班前必须提交分支，并push到远程。
-   hotfix、feature、bugfix、refactor分支尽量按照功能点或修复重构的问题及时commit（不要求push）

## git操作规范

![0.png](file:///Users/apple/.config/joplin-desktop/resources/0cc9b1437a754eee95d41ec3e9401cef.png?t=1672707217732)

0.png

## 重点注意事项

### 在团队开发中，git的操作流程应至少遵循如下步骤（重点）

-   commit 本地代码到本地库（防止出现问题时，可以回调）
-   pull 远程库的代码
-   push 本地库代码到远程看

### 在合并分支流程，如：分支1的代码需同步到分支2上

-   先切换到需要同步代码的分支2
    
    ![1.png](file:///Users/apple/.config/joplin-desktop/resources/57562b9546ce4c28b3f374a7c14bc9a1.png?t=1672707416877)
    
    1.png
    
    ![2.png](file:///Users/apple/.config/joplin-desktop/resources/d325b61ff0e74663b8dee20096194dcd.png?t=1672707423589)
    
    2.png
    
-   点击需要同步过来的分支1，点击merge into current
    
    ![3.png](file:///Users/apple/.config/joplin-desktop/resources/9439f7b0c7bb47419e1c098d983db9c5.png?t=1672707445816)
    
    3.png
    

### 如果pull代码过程中，merge出错的处理方式

-   通过 git reflog [分支名] 的命令查看响应的日志
    
    ![4.png](file:///Users/apple/.config/joplin-desktop/resources/454a4a26b0e9436b829d1c32a205db6d.png?t=1672707489984)
    
    4.png
    
-   获得上一次 commit_id 如上：afe54ca，然后通过 git reset --hard commit_id，例如：git reset --hard afe54ca
    
    ![5.png](file:///Users/apple/.config/joplin-desktop/resources/a1b729a0348447a696feb48fb0a6a4c8.png?t=1672707510642)
    
    5.png
    
-   再通过pull 命令拉取最新的代码，再重新合并即可
    

### 重点注意

-   每次 pull 前，切记要提交本地代码到本地库中，否则可能回出现合并代码出错，导致代码丢失的情况