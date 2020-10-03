# yapkg 通用包管理工具
[TOC]
## 实现目标
### 【M1】用户授权系统
- 基本的基于用户名和密码的授权验证【H】
- 需要绑定并验证邮箱【M】
- 基于 `appid, appsecret` 的授权 api 接口【H】
### 【M1】实现基本包管理功能
- 管理软件包（下简称包 / pkg），包括 CRUD
  - 只允许用户对自己操作范围内的包内容进行编辑
  - 暂时用户自己只能管理自己的包，例如 `tamce/demo_pkg`，类似 github 的 repo 持有机制
  - 后续需要类似 github，允许用户创建命名空间（组织），并在对应的命名空间下发布内容，权限控制需要至少区分几个等级，需要预先保留好拓展性【M3】
- 一个包的元信息（随便参照一个包管理工具），建议抽出一层独立于实际表示方式的接口，下层再实现具体的解析器（例如 json / yaml / http / git 等）
  - 来源（默认即为 `yapkg`），可暂时不实现，但需要预留类似的接口能力
  - 所属领域（既大分类，例如 `java`, `php` etc）【M3】这个是实现通用包管理的关键，也是日后接入 GUI 搜索时需要参与搜索的字段，但这个内容本身不应该出现在包的元信息里，而是在调用接口时做区分并保存到数据库记录
  - 名称（命名空间+名称），如 `tamce/demo_pkg`
  - 版本，符合 semver 2.0 规范
  - 作者信息
  - 依赖的包 (requires)，如 `tamce/another_pkg: ^1.0`
  - 开发环境依赖的包 (`requires_dev`)
  - 其他待定待拓展

### 【M1】实现 API 接口交互
- 注册（可暂不实现，内测期间不开放注册）
- 登录
- 包元信息的 CRUD；元信息接口和实体内容需要保持一致性
- 包实体内容（例如代码 / jar 包等形式）的上传和下载接口

### 【M2】实现 Web GUI 交互界面
### 【M2】实现 webhook / github 触发动作
### 【M3】实现可切换领域

## 开发规范
1. 所有开发均在 dev 分支进行（或者专门为每一个功能开发开新的功能分支）
2. 每一个功能性内容（即使很小）均需要提 issue
3. 当某个 issue 被解决，需要选择对应 commit 发起 pr 到 master
4. pr 等待 code review 后则可以被合入 master
5. commit 信息的首行要以 `subject(module): topic` 的格式进行，其中 module 可省，例如
   1. `doc: add target for M1-M3`
   2. `feat(api): complete user auth api`
   3. `fix(api): fix user logout api bug`
   4. `misc: fix spelling issue in readme`
   5. 其实都可以写中文（（