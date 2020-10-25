# yapkg 数据库定义
- [yapkg 数据库定义](#yapkg-数据库定义)
  - [说明](#说明)
  - [users 用户表](#users-用户表)
  - [applications 应用表](#applications-应用表)
  - [namespaces 命名空间表](#namespaces-命名空间表)
  - [user_namespace 命名空间成员关系表](#user_namespace-命名空间成员关系表)
  - [packages 包表](#packages-包表)
  - [user_package 包成员关系表](#user_package-包成员关系表)
## 说明
* 所有的 `updated_at`, `created_at` 针对的都是 “数据库记录” 的创建和更新时间，因此需要在插入数据/更新数据时对应更新对应的字段。这个习惯/约定是来自于一些其他 ORM 框架。
## users 用户表
* 暂时先不考虑接入 github 等第三方登录
```yaml
id: primary
username: varchar(63), index
password: char(63)
email: varchar(255), index
status: enum { kPending, kActive, kForbidden } # 暂时先三种，未激活，已激活，已禁用
created_at: timestamp
updated_at: timestamp
```
## applications 应用表
```yaml
id: primary
user_id: (user_id)
appid: varchar(63), index # 暂时 length 先 63
secret: varchar(63)
type: enum { kReadOnly, kPublish, kAll }, index
status: enum { kEnabled, kDisabled }, index # 用于 disable 而不删除
description: varchar(127) # 用户添加的注释描述
created_at: timestamp
updated_at: timestamp # type / status 等基本属性修改的时间
used_at: timestamp # 最后一次使用的时间
```
## namespaces 命名空间表
* 创建用户时自动创建一个同名的 `namespace`，因为 `foo/bar` 中的 `foo` 既可能是用户，又可能是命名空间，如果不创建，那查询时也需要额外查询用户表，因此自动创建用户同名的命名空间用于简化逻辑。
```yaml
id: primary
owner_id: (user_id), index
name: varchar(63), index
created_at: timestamp
updated_at: timestamp
```
## user_namespace 命名空间成员关系表
```yaml
id: primary
namespace_id: (namespace_id), index
user_id: (user_id), index
# TODO: 详细定义 role 的权限，可以先只实现 Owner 和 Member 的区别
role: enum { kOwner, kManager, kMember, kReadOnly }, index
created_at: timestamp
updated_at: timestamp
```
## packages 包表
```yaml
id: primary
field: varchar(63), index # 包所属的“领域”, eg. npm/composer/qqbot etc.
namespace_id: (namespace_id), index
name: varchar(63), index
full_name: varchar(127), index # eg. "foo/bar"，因为通过 namespace/package 访问的频率很高，所以这里冗余存储，否则需要联表查询 namespace 表，一般情况下也是不允许修改 namespace 的名字的
version: varchar(31) # latest version
description: varchar(127)
type: varchar(63), index # eg. library, plugin etc. 先不管，当作一个平凡字符串的属性就可以，我想法是可能用于搜索时的选择
# 暂时不存 meta path，先用 “约定” 的位置
published_by: (user_id)
published_at: timestamp
created_at: timestamp
updated_at: timestamp
```
## user_package 包成员关系表
* TODO 先不做，先只做 namespace 粒度的权限控制
