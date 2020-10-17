# yapkg-backend 接口文档
- [yapkg-backend 接口文档](#yapkg-backend-接口文档)
  - [通用约定](#通用约定)
  - [Todos](#todos)
  - [用户认证系统](#用户认证系统)
    - [用户名-密码注册](#用户名-密码注册)
    - [用户名-密码登录](#用户名-密码登录)
    - [登出](#登出)
    - [获取当前用户的信息](#获取当前用户的信息)
    - [获得某一用户信息](#获得某一用户信息)
    - [修改当前用户信息](#修改当前用户信息)
    - [生成用户appid-appsecret对](#生成用户appid-appsecret对)
    - [获得用户当前持有的appid-appsecret对](#获得用户当前持有的appid-appsecret对)
    - [删除用户appid-appsecret对](#删除用户appid-appsecret对)
  - [包管理](#包管理)
    - [创建新包](#创建新包)
    - [获取一个包的信息](#获取一个包的信息)
    - [搜索符合条件的包的简要信息](#搜索符合条件的包的简要信息)
    - [获取一个命名空间下所有包的简要信息](#获取一个命名空间下所有包的简要信息)
    - [删除一个已经存在的包](#删除一个已经存在的包)
    - [为一个包发布新版本](#为一个包发布新版本)
    - [为一个包添加新的合作者](#为一个包添加新的合作者)
    - [获得一个包的所有合作者](#获得一个包的所有合作者)
    - [修改一名合作者的角色](#修改一名合作者的角色)
    - [将一名用户从包的合作者中移除](#将一名用户从包的合作者中移除)
  - [命名空间管理](#命名空间管理)
    - [创建新的命名空间](#创建新的命名空间)
    - [获取命名空间信息](#获取命名空间信息)
    - [删除命名空间](#删除命名空间)
    - [为命名空间添加新的成员](#为命名空间添加新的成员)
    - [修改命名空间成员的角色](#修改命名空间成员的角色)
    - [移除命名空间的成员](#移除命名空间的成员)

## 通用约定
* 所有返回内容均为`json`
* 正常响应均返回`200`状态码
* 不说明返回数据具体形式的默认只返回请求状态
* 未进行用户验证/用户验证失败返回`401`
* 当前帐号/`token`权限不足返回`403`
* 参数错误返回`400`
* 资源不存在返回`404`
* 用错误信息区分更细分的错误情况
* 使用`token`代替`session`进行用户验证时，请设置请求头中的`X-Auth-Token`字段为`token`值
* 利用`client_id`以及`secret`生成`token`的方式：
  1. 序列化请求体并对其进行URL安全的`Base64`编码
    ```js
        serialized = urlsafe_base64_encode(serialized_requestBody)
    ```
  2. 生成一个UNIX时间戳（单位秒）`timestamp`，和一个128位随机数`random`（都需要进行URL安全的`Base64`编码）
  3. 将`serialized`、`timestamp`和`random`用英文冒号`:`进行拼接，得到待签名字符串
    ```js
        unsignedStr = "<timestamp>:<random>:<serialized>"
    ```
  4. 使用`secret`，计算待签名字符串`unsignedStr`的`HMAC-sha1`签名，并对结果进行URL安全的`Base64`编码
    ```js
        signature = urlsafe_base64_encode(hmac_sha1(unsignedStr, "<secret>"))
    ```
  5. 将`client_id`、`signature`、`<unsignedStr>`用英文冒号`:`顺次连接起来，即得到可以使用的`token`
    ```js
        token = "<client_id>:<signature>:<unsignedStr>"
    ```
* 用户信息的结构如下。未经特殊说明，所有返回用户完整信息的接口都会返回这一结构的信息。
  ```js
    {
        userid: "userid",
        username: "username",
        email: "example@example.com",
        avatar_uri: "avatar_uri"
    }
  ```
* 包的元信息的结构如下。未经特殊说明，所有返回包完整元信息的接口都会返回这一结构的信息。
  ```js
    {
        meta: {    
            name: "namespace/package_name",
            version: "x.y.z[-先行版本号]",
            dependencies: {
                ...
                dependency_n: "x.y.z",
                ...
            },
            dev_dependencies: {
                ...
                dependency_n: "x.y.z",
                ...
            },
            type: "PUBLIC",
            description: "this is a package"
        }
        package_id: "package_id",
        namespace_id: "namespace_id",
        field: "java",
        publisher: {
            user_id: "user_id",
            username: "username"
        },
        last_updated: "yyyy-mm-dd",
    }
  ```
* 命名空间信息的结构如下。未经特殊说明，所有返回命名空间完整信息的接口都会返回这一结构的信息。
  ```js
    {
        namespace_id: "namespace_id",
        namespace: "namespace_name",
        owner: "owner_name",
        member: [
            ...
            {
                username: "username",
                role: "role",
                avatar_uri: "avatar_uri"
            },
            ...
        ]
    }
  ```
* 该API文档只涉及要放入平台DB的部分
    
## Todos
1. 邮箱验证（生成带激活码的验证链接，点击链接后验证激活码并令激活码过期）
2. 在危险操作（例如删除）之前是否需要用户输入密码或重复包/命名空间的全名以进入`sudo mode`？
3. 在文件上传时进行token生成，还对整个请求体进行签名吗？
4. HMAC-sha1足够安全吗？

## 用户认证系统
### 用户名-密码注册
```js
POST /register
{
    username: "username",
    password: "password",
    email: "example@example.com"
}

// 注册完成，返回200
// 参数错误/缺少参数/用户名存在/邮箱存在，返回400
```
返回非敏感的所有用户信息

### 用户名-密码登录
```js
POST /login
{
    username: "username",
    password: "password"
}

// 注册完成，返回200
// 参数错误，返回400
// 用户名不存在或与密码不匹配，返回401
```
返回登录用户的用户信息

### 登出
```js
GET /logout
```

### 获取当前用户的信息
```js
GET /me
```
返回当前用户的基本信息

### 获得某一用户信息
```js
GET /users/{username}
```
返回指定用户的信息

### 修改当前用户信息
```js
PATCH /me
{
    user_id: "user_id",
    username: "username",
    password: "password",
    email: "example@example.com",
    avatar_uri: "avatar_uri",
}
```
返回修改后当前用户的信息

### 生成用户appid-appsecret对
用户需要根据申请的`client_id`和`secret`生成`token`进行使用。
```js
POST /applications/clients
{
    type: "READ_ONLY", // READ_ONLY or PUBLISH or ALL
    expires: 1602923788 // UNIX时间戳表示的过期时间，单位秒，为0表示永久
}
```

### 获得用户当前持有的appid-appsecret对
```js
GET /applications/clients
```
返回用户所有的`client_id`-`secret`对
```js
[
    ...
    {
        client_id: "client_id",
        secret: "secret",
        type: "READ_ONLY",
        expires: 1602923788
    },
    ...
]
```

### 删除用户appid-appsecret对
用户应该定期更新其使用的`client_id`-`secret`对。
```js
DELETE /applications/clients/{client_id}
```

## 包管理

### 创建新包
`namespace`为一个有效的命名空间名称。用户可以在自己的命名空间（自己的用户名）下创建新包。获得某一命名空间的`MEMBER`角色的用户可以在这一命名空间下创建新包。类型为`PRIVATE`的包只有合作者（collaborator）才可以访问。
CLI应该将整个包目录中的所有文件打包成单个`.tgz`文件进行上传。
```js
POST /namespaces/{namespace}/packages
Content-Type: multipart/form-data; boundary=<boundary>

--<boundary>
Content-Disposition:    form-data; name="file"; filename="<filename>"
Content-Type: application/octet-stream
Content-Transfer-Encoding: binary

...binary data...
--<boundary>--

// 成功创建包，返回200
// 参数错误（缺少参数/包名冲突/namespace id不存在），返回400
// 未认证返回401
// 在指定的namespace下没有创建包的权限，返回403
```
创建成功返回新创建的包的完整信息

### 获取一个包的信息
```js
GET /namespaces/{namespace}/packages/{package_name}
```
返回指定的包的完整信息


### 搜索符合条件的包的简要信息
类型为`PRIVATE`的包不会被搜索出来
```js
GET /packages?query={keyword}[&page={page}]
// query参数：搜索关键词
// page参数：可选，说明请求第几页的数据，页数上溢返回最后一页，页数下溢返回第一页，缺省值为1
```
返回根据条件搜索得到的所有包的简要信息
```js
[
    ...
    {
        meta: {
            name: "namespace/package_name",
            version: "x.y.z",
            description: "this is a package"
        }
        package_id: "package_id",
        namespace_id: "namespace_id",
        field: "java",
        publisher: {
            user_id: "user_id",
            username: "username",
        },
        last_updated: "yyyy-mm-dd"
    },
    ...
]
```


### 获取一个命名空间下所有包的简要信息
```js
GET /namespaces/{namespace}/packages[?page={page}]
// page参数：可选，说明请求第几页的数据，页数上溢返回最后一页，页数下溢返回第一页，缺省值为1
```
返回`namespace_id`指定命名空间下的所有包的简要信息
```js
[
    ...
    {
        meta: {
            name: "namespace/package_name",
            version: "x.y.z",
            description: "this is a package"
        }
        package_id: "package_id",
        namespace_id: "namespace_id",
        field: "java",
        publisher: {
            user_id: "user_id",
            username: "username",
        },
        last_updated: "yyyy-mm-dd"
    },
    ...
]
```

### 删除一个已经存在的包
```js
DELETE /namespaces/{namespace}/packages/{package_name}
// 成功删除包返回200
// 不存在这样的包返回204 No Content
// 未认证返回401
// 不具有删除该包的权限返回403
```

### 为一个包发布新版本
```js
PUT /namespaces/{namespace}/packages/{package_name}
Content-Type: multipart/form-data; boundary=<boundary>

--<boundary>
Content-Disposition:    form-data; name="file"; filename="<filename>"
Content-Type: application/octet-stream
Content-Transfer-Encoding: binary

...binary data...
--<boundary>--

// 成功发布包，返回200
// 参数错误（缺少参数/package id不存在），返回400
```
发布新版本成功返回新版本包的完整信息

### 为一个包添加新的合作者
```js
POST /namespaces/{namespace}/packages/{package_name}/collaborators
[
    ...
    {
        username: "username",
        role: "role"
    }
    , ...
]
```

### 获得一个包的所有合作者
```js
GET /namespaces/{namespace}/packages/{package_name}/collaborators
```
返回合作者的信息
```js
[
    ...
    {
        collaborator: {
            user_id: "user_id",
            username: "username",
            avatar_uri: "avatar_uri",
        },
        role: "role"
    },
    ...
]
```

### 修改一名合作者的角色
```js
PATCH /namespace/{namespace}/packages/{package_name}/collaborators/{username}
{
    role: "role"
}
```

### 将一名用户从包的合作者中移除
```js
DELETE /namespace/{namespace}/packages/{package_name}/collaborators/{username}
```

## 命名空间管理
### 创建新的命名空间
注意命名空间不应该和已经存在的用户名重复，并且如果调用类似于`/namespace/{username}`，应该返回`404`。
```js
POST /namespace
{
    namespace: "namespace_name",
}
```
创建成功返回新建的命名空间信息

### 获取命名空间信息
```js
GET /namespace/{namespace}
```
返回指定的命名空间信息

### 删除命名空间
会将命名空间中的所有包一并删除
```js
DELETE /namespace/{namespace}
```

### 为命名空间添加新的成员
```js
POST /namespace/{namespace}/members
{
    ...
    {
        username: "username",
        role: "role"
    },
    ...
}
```

### 修改命名空间成员的角色
```js
PATCH /namespace/{namespace}/members/{username}
{
    role: "role"
}
```

### 移除命名空间的成员
```js
DELETE /namespace/{namespace}/members/{username}
```
