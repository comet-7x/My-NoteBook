## 📚 GitHub 仓库的分支命名规范与类型

一个清晰、一致的分支命名规范是多人协作和项目管理的关键。它能让团队成员快速了解分支的用途、类型和状态。

### 1. 分支类型（Branch Types）

主流的 Git 工作流（如 Git Flow 或相似的模式）通常将分支划分为以下几种主要类型：

| **类型**    | **命名约定**                  | **目的**                                                                                      |
| --------- | ------------------------- | ------------------------------------------------------------------------------------------- |
| **主分支**   | `main` / `master`         | **生产环境代码**。永远保持稳定、可部署的状态。所有发布版本都应基于此分支。                                                     |
| **开发分支**  | `develop` / `development` | **集成分支**。所有新功能完成后合并到此分支，进行集成测试。它是日常开发的基线。                                                   |
| **功能分支**  | `feature/`                | 开发**新功能**。从 `develop` 分支派生，完成后合并回 `develop`。                                                |
| **修复分支**  | `bugfix/`                 | 修复**`develop` 分支上的 bug**。从 `develop` 派生，完成后合并回 `develop`。                                   |
| **热修复分支** | `hotfix/`                 | 修复**`main/master` 分支上的生产环境 bug**。从 `main/master` 派生，完成后需要合并回 `main/master` **和** `develop`。 |
| **发布分支**  | `release/`                | **准备新版本发布**。从 `develop` 派生，只进行小幅 bug 修复、版本号更新等。完成后合并回 `main/master` **和** `develop`。        |

---

### 2. 分支命名规范（Naming Conventions）

一个良好的命名规范通常包括**类型**、**用途/描述**，有时还包括**来源**或**关联 ID**。

#### A. 规范结构（推荐）

推荐的结构是使用斜杠 `/` 来分隔类型和描述，使其更具可读性：

$$\text{branch-type}/\text{description-or-ID}$$

**斜杠的使用是关键，它使得 Git 命令（如 `git branch --list 'feature/*'`）更容易进行分组和筛选。**

#### B. 详细命名示例

|**分支类型**|**推荐命名格式**|**示例**|**说明**|
|---|---|---|---|
|**功能**|`feature/[short-description]` 或 `feature/[issue-id]-[description]`|`feature/user-profile-page`|开发用户资料页。|
|||`feature/ISSUE-100-login-oauth`|关联到 Issue #100，实现 OAuth 登录。|
|**热修复**|`hotfix/[short-description]` 或 `hotfix/[issue-id]-[description]`|`hotfix/fix-payment-bug`|紧急修复生产环境的支付 bug。|
|||`hotfix/CRITICAL-45-api-timeout`|关联到 Issue #45，修复 API 超时问题。|
|**修复**|`bugfix/[short-description]` 或 `bugfix/[issue-id]-[description]`|`bugfix/sidebar-alignment`|修复开发分支上的侧边栏对齐问题。|
|**发布**|`release/[version-number]`|`release/v1.2.0`|准备发布 1.2.0 版本。|
|**重构/优化**|`refactor/[description]`|`refactor/split-large-service`|重构，拆分大型服务模块。|

#### C. 命名通用规则

1. **使用小写字母：** 保持一致性，避免在不同操作系统上可能出现的区分大小写问题。
    
2. **使用连字符 `-` 或下划线 `_`：** 用作单词之间的分隔符。推荐使用**连字符 `-`**，它在 URL 中更常见，可读性更强。
    
3. **简洁但有意义：** 描述要足够清晰，让其他团队成员一看就知道分支是做什么的，但不要太长。
    
4. **避免特殊字符：** 仅使用字母、数字、连字符（-）、下划线（_）和斜杠（/）。
    
5. **避免版本号（除非是发布分支）：** 功能或 bugfix 分支不应该包含版本号，因为它们可能会在多个版本中存在。