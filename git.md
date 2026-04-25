Git 项目完整生产工作流（新手通用·企业标准）

本文整合从零搭建 Git 仓库、分支管理、开发、提交、合并全套规范，适配所有前后端项目，解决新手常见报错，为可直接落地的**团队生产标准流程**。

## 一、仓库初始化流程

适用于本地新项目，首次接入 Git 版本控制

### 1. 基础初始化命令

```bash
# 初始化本地git仓库
git init

# 添加当前目录所有文件到暂存区（禁止使用 git add ..）
git add .

# 提交代码到本地仓库，编写备注
git commit
```

### 2. 新手常见问题解决

- **报错：..: '..' is outside repository** 原因：`..` 代表上级目录，Git 仅允许管理仓库内部文件 解决方案：统一使用`git add .` 提交当前仓库所有文件
- **警告：LF will be replaced by CRLF** 原因：Windows 与 Mac/Linux 换行符差异，仅为警告，不影响项目运行 永久解决：执行全局配置 `git config --global core.autocrlf true`
- **命令拼写错误：comit** 正确命令：`git commit`

## 二、主分支配置规范

### 1. 主分支作用

主分支（默认 `main/master`）为**生产稳定分支**，仅存放可上线、无BUG的成品代码，**禁止直接在主分支开发代码**。

### 2. 主分支改名（解决中文引号异常）

针对分支名自带中文引号、常规改名命令失效的场景，使用最简改名命令：

```bash
# 直接修改当前所在分支名称（无需填写旧分支名）
git branch -m 新分支名
```

### 3. 全局默认分支配置（一劳永逸）

```bash
# 后续所有新建git仓库，默认主分支为master
git config --global init.defaultBranch master
```

## 三、分支创建与开发规范（核心）

### 1. 分支来源规则

**新分支永远从【最新的主分支】创建**，保证开发代码基于稳定版本，避免兼容BUG。 核心逻辑：当前处于哪个分支，新分支就复制哪个分支的代码。

### 2. 分支分类（企业通用）

- **master/main**：生产主分支，稳定上线代码
- **dev**：开发测试分支，整合所有功能代码
- **feature/xxx**：新功能分支（例：feature/login、feature/cart）
- **bugfix/xxx**：BUG修复分支（例：bugfix/login-error）

### 3. 分支操作完整命令

```bash
# 1. 切回主分支，保证代码最新
git checkout master

# 2. 仅创建分支，不切换
git branch dev

# 3. 创建分支并直接切换（最常用）
git checkout -b feature/cart

# 4. 新版git简写命令（推荐）
git switch -c bugfix/order

# 5. 查看所有本地分支
git branch

# 6. 切换已有分支
git switch 分支名
```

## 四、日常开发提交流程

所有代码编写、修改，统一在自定义功能分支完成，遵循以下闭环流程：

```bash
# 1. 修改代码后，添加文件至暂存区
git add .

# 2. 提交代码，填写清晰备注
git commit -m "完成购物车新增商品功能"

# 3. （可选）拉取主分支最新代码，解决版本冲突
git checkout master
git pull
git switch 功能分支
git merge master

# 4. 功能开发完成，合并到开发分支/主分支
git checkout dev
git merge 功能分支
```

## 五、远程仓库同步流程（拓展）

本地仓库搭建完成后，可关联 Gitee/GitHub 远程仓库，实现代码备份与团队协作：

```bash
# 1. 关联远程仓库
git remote add origin 远程仓库地址

# 2. 推送本地分支到远程
git push -u origin 分支名

# 3. 删除远程旧分支
git push origin --delete 旧分支名
```

## 六、Git 开发黄金准则（必遵守）

1. 主分支仅保存稳定可上线代码，**严禁直接开发**
2. 所有新功能、BUG修复，必须新建独立分支
3. 所有功能分支，统一从最新主分支创建
4. 每次提交代码，必须填写清晰、简洁的提交备注
5. 开发完成后合并分支，废弃分支可及时删除，保持仓库整洁