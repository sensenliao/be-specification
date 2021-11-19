# 规范约束
## 业务说明

indicator 名为指标，在项目的数据表设计中，该名称对应的数据表存储的是指标树的一个一个节点以及对应所属关系（pid），存在树型的层级结构，树型的每一层都是和业务紧密关联的：根 → 类别 → 维度 → 指标 → 变量，主要有这几个层次级别，其中 `指标` 这两个字就会经常出现歧义，如 指标id 可以表示指标表任意一条数据的id，也可以表示指定层级的指标的id；所以在项目的源码注释或是沟通上经常可以看见这样的描述，`指标 - 指标`、`指标（狭义）` 来特指指标层面的指标节点

目前项目数据表中 indicator、ranking_type 存储的数据结构是树状结构，树状结构有如下特征

- pid

- path

  ```go
  // 应该实现的方法，这样在调用的代码逻辑中就可以方便的根据 path 获取指定级别的父级id了
  func (x Xxx) GetPath() help.Path {
      return help.Path(x.Path)
  }
  ```

- level

## 代码风格

go 中不推荐空格的源码编排风格，go fmt 命令会将所有的空格的编排风格转换为 tab

- 注释 `\\` 和注释内容之间都应该有一个空格
- `conf` 配置文件中支持的注释语法有两种 `#` `;`，约定前者为大模块注释，后者为小模块注释

## 目录结构

|                       | beego 生成 | 说明                                                         |
| --------------------- | ---------- | ------------------------------------------------------------ |
| conf                  | true       |                                                              |
| routers               | true       |                                                              |
| docs                  | true       |                                                              |
| tests                 | true       |                                                              |
|                       |            |                                                              |
| controllers           | true       |                                                              |
| controllers/internals | false      | 一些内部的，不提供给前端开发人员的接口。如果数据维护、刷新缓存等 |
|                       |            |                                                              |
| models                | true       | 最简单纯净的数据表实体                                       |
| models/internal       | false      | 项目中定义的能够根据数据表自动生成 Go 数据实体源码的 Groove 脚本，生成的数据实体 |
| models/const          | false      | 常量、枚举；数据表中特定业务含义数据的指定字段值等           |
| models/bo             | false      | 带有复杂业务逻辑处理方法的结果集实体，即使会直接作为结果集返回，也不应该放到 dto 中 |
| models/dto            | false      | 接口结果集实体（包的循环依赖：不定义实体转换方法）           |
| models/form           | false      | 接口参数实体                                                 |
|                       |            |                                                              |
| repo                  | false      | DAO                                                          |
|                       |            |                                                              |
| usercase              | false      | 业务处理                                                     |
|                       |            |                                                              |
| module                | false      | 独立模块（不含 dao 的数据交互操作，如果含有，整个模块应该放到 usercase 下） |
| filter                | false      | 拦截器                                                       |
| help                  | false      | 工具类                                                       |

## 部署更新

传统项目部署的痛点有，构建的二进制包体积大，构建时间长 + 上传时间长、项目运行环境多（不同环境的配置文件，具体参数项，差异较大），但是目前均未有这样的问题，故项目的部署更新模式，还是以全过程手动操作为主。

服务器上的项目部署更新，要求有一个好习惯，就是将一定时间内的二进制程序包，保留下来，以应对新程序包出问题时，能够立马回滚，保证服务的可用。

综上，参考当前项目中的 `buildupx.bat` 构建脚本（Windows 操作系统下，可直接运行）

```
@echo off
set proj_name=项目名
set ts=%date:~2,2%%date:~5,2%%date:~8,2%.%time:~0,2%%time:~3,2%%time:~6,2%
set ts=%ts: =0%
set exe_name=%proj_name%-%ts%.exe
set upx_name=%proj_name%-%ts%-upx.exe
go build -o %exe_name% && upx -9 -o %upx_name% %exe_name% && rm %exe_name%
```

注意，在最后构建，还做了一个压缩，`upx` 是一个开源的二进制文件压缩工具

## 命名规范

### 慎用 `test`

用于代表测试代码的源码文件 `xxx_test.go` 中定义的公共变量、常量，其他源码文件无法获取引用

`beego` 中包名为 `test` 的包，中的 Controller 会被忽略

### 文件命名（业务）

指标表 狭义指标 id：indLevelId

指标表 狭义变量 id：indLevelVarId

变量表 变量 id：indVarIds | varIds

### 文件命名

包名所对应的含义应该利用好，即不应该有如：`包名_文件名.go` 或 `文件名_包名.go`

### 变量命名

**Receiver（层）**

c *XxxController、u *XxxUsercase、r *XxxRepo

**Receiver（表单实体）**

f *Xxx、f *QXxx

**SQL**

q := ...

b := strings.Builder{}

### 结构体命名

除了三大层 `XxxController`、`XxxUsercase`、`XxxRepo`，其他例如 `form` 包下 和 `dto` 包下的结构体类型都不应该再带有后缀了

参数实体：`Xxx`、`QForm`

### 通道变量名

仅用作信号通道，传输的数据没有实际含义时，命名为：`xxxSignal`

通道传输的数据有实际含义时，命名为：`xxxChannel`

### 枚举类型命名

在项目结构中了解到，静态概念都放在 `models/const` 包下，但是 `const` 同时也是 go 的关键字，综合考虑，决定采用包名表示枚举含义，所有具体的枚举类型都叫 `Enum`

**举例**

```go
package gender

type Enum int

const (
    Female Enum = 0
    Male   Enum = 1
)
```

```go
package fruit

type Enum struct {
  Code int
  Name string
}

var (
  Apple      = Enum{code: 1, name: "苹果"}
  WaterMelon = Enum{code: 2, name: "香蕉"}
  // ...
)

func (e Enum) String() string {
    return string(e)
}

func (e Enum) Valid() bool {
    switch e {
        case Apple, WaterMelon, ...:
        	return true
        default:
        	return false
    }
}
```

### 方法命名

#### DTO

一般，repo 层查询的数据含有 json 结构的数据，需要反序列化；或是包含一些数据操作处理，那应该在 dto 实体所在源码文件中定义 Parse 方法进行处理，见下边定义

 `(d *dto.Xxx) Parse(*models.Xxx)`

``(d *dto.Xxx) Parse()`

`ParseXxxs([]*models.Xxx)[]*dto.Xxx`

`ParseXxxs([]*dto.Xxx)`

#### DAO

> 本着 `usercase` 层方法的方法名可以简化承载业务含义，而 repo 层方法仅义如名，一般不含业务含义的大体方向；但若方法确实包含比较重的业务含义也就不应该按照下面的规则，起一个业务含义名称即可

`<Find | List> [Part | DTO] By XxxAnd...`

（条件一般不超过两个，超过两个需要适当将条件单词简写，以避免方法名过长）

**说明**

Find 或 List 分别代表查询一个或列表，允许 `ById` 简写

Part 或 DTO 代表查询的内容，不写代表查全了

**举例**

查询一个：`ById`、`FindPartById`、`FindDTOById`

查询列表：`ListByIds`、`ListPartByIds`、`ListDTOByIds`

> `beego orm` 提供操作方法，方法名为：`Insert`、`Update`、`Delete`

保存或更新：`Upsert`

保存：`InsertOne`

更新：`UpdateOne`

**拓展**

服务层要建立接口实现的层次结构，`dao` 视情况而定，如果和业务强相关的查询多了，就设定接口层次

#### 树型节点

关于树型结构数据节点的通过 id 查询一条数据的方法（usercase 层面），FindById 方法可以添加一个可变参数的形参，当参数不为空时，就应该在方法中查询后，进行校验

`FindById(id int64, expectLevel ...int) (*models.Xxx, error)`

### 三方对接相关

const：常量

models：参数、响应 实体

i_xxx：每个接口对接方法单独划分开来

## 接口规范

### Restful

- GET

  QUERY

- POST

  JSON（不推荐 FormData）

  前端的 axios 库、各大厂商的开放 api，接口参数默认都是 JSON 格式

  与其用内部约定好的数据格式（逗号隔开的字符串），就不如使用通用的 JSON 强类型格式来的实在

- PATCH

  JSON - 更新部分字段：如，状态

- PUT

  JSON - 如果 创建 和 更新 接口是分开的，POST 用于创建、PUT 用于更新

- DELETE

  PATH

### 参数校验

仅限于开发阶段会出现的错误，参数非空提示：`param xxx can not be empty`，前端应该根据该提示，了解到哪些参数是必填的，只有有值才能发起请求；而不是，直接将响应的英文作为用户的提示

推荐在符合以下条件的接口参数使用参数实体，结合 `beego` 的 `validation.ValidFormer`，配置校验解析方法去做参数初步处理

不推荐使用 `form` 标签去做校验，因为设计理念不合理，提示的字段名是错的

- 在接口参数预计会大于 3 个

- 有较为复杂的基本校验规则时（如，数字需要限定在一定范围内、电话号码等）

- 参数需要初步解析才能使用（如，base64 编码、内容为逗号隔开的多个元素字符串等）

> 目前 `BaseController` 提供了 `ParseFormAndValidate` 和 `ParseJsonAndValidate` 方法

### 查询条件（分页）

先列举两个实际业务场景中的参数实例

1. 分页参数
2. 搜索条件：多个字段，空格隔开
3. 动态响应条件字段：多个字段，英文逗号隔开

**样例**

直接将项目中源码贴出来；

需要特别提及的是，`strings.Split("", " ")` 返回的是 `[""]（len = 1）`，所以非空判断是为了特意保持 nil 值

如果传参方式是 `formData` 则标签是 `form`；如果传参方式是 `json`，则标签是 `json`

```go
package form

import (
	"github.com/beego/beego/v2/core/validation"
	"rankingbase-api/help"
	"strings"
)

type IndVarDetailRoot struct {
	Search        string `form:"search"`        // 逗号隔开
	Include       string `form:"include"`       // 需要接口返回的额外信息；include=indVar,detailDef 则返回对应的变量信息，及明细样式定义
	Limit         int    `form:"limit"`         // 每页数据量（默认10）
	Page          int    `form:"page"`          // 页码（默认1）

	SearchFields  help.StrSlice
	IncludeFields help.StrSlice
}

func (f *IndVarDetailRoot) Valid(v *validation.Validation) {
	if f.Limit == 0 {
		f.Limit = DefaultLimit
	}
	if f.Page == 0 {
		f.Page = DefaultPage
	}
	if f.Limit < 1 {
		_ = v.SetError("limit", "param limit illegal")
	}
	if f.Page < 1 {
		_ = v.SetError("page", "param page illegal")
	}

	if len(f.Search) != 0 {
		f.SearchFields = strings.Split(f.Search, " ")
	}
	if len(f.Include) != 0 {
		f.IncludeFields = strings.Split(f.Include, ",")
	}
}
```

## 接口文档

### 大纲结构

- 页面列表

  页面 X1：需要调用的接口...，链接到接口列表

  ...

- 接口 A

  业务场景说明：描述清楚业务背景、注意事项等

  接口地址：首先，标注清楚请求方式；然后，如知道具体部署环境的接口地址，就把每一项列举清楚；如不知道，给出相应的路由地址即可，如：`[GET] /v1/user`

  参数说明（名称、类型、位置、是否必须、备注）

  参数样例

  响应说明（名称、类型、描述）

  响应样例

- 接口 B

  ...

- ...

- 其他

  如业务中有进行三方对接，就贴出所有三方相关的文档地址链接

### 业务迭代

当产品提出新的需求，业务进行迭代，除了更新主接口文档，还应该创建编写一份变更文档来进行说明

### 总结

本文档不是最终形式，会持续更新

部门暂无严格要求，文档旨在将业务和每个接口设计说清楚，如果文档能以一定的结构阐述清楚，以上规则可无视

## 数据表设计

### ORM 映射
首先提及一个 `GROOVY` 脚本文件 `Copy as Go struct.groovy`（由关关创建编写和维护）

**作用**：将数据库表作为脚本的参数从而可以得到对应的 beego 的 orm 实体定义

**脚本使用**：将脚本文件放置到 Goland 项目的 `(a)` 目录下，然后通过 Goland 自带的 DB 连接工具，连接到指定的数据库，通过 `(b)` 操作应用运行脚本

> (a) /Scratches and Consoles/Extensions/Database Tools and SQL/schema/Copy as Go struct.groovy

> (b) 框选想要操作的表 → 右键 → Scripted Extensions → Copy as Go struct.groovy → 剪切板中就存有了 orm 实体定义

**具体规则**

| 维度                   | 概念                                                         |
| ---------------------- | ------------------------------------------------------------ |
| 所有键：主键id、关联键 | GO 类型：int64、MySQL 类型：int                              |
| 字段是否为空           | GO 语言中所有基础数据类型都有零值，所以当要表示无意义的空值时，应将字段类型改为特定类型对应的指针类型 |
| 枚举                   | 得到枚举名称 → SELECT xxx<br />得到枚举序号 → SELECT xxx + 0 AS xxx |

[beego]: https://github.com/shanghairanking/docs/blob/main/3.%E6%95%B0%E6%8D%AE%E5%BA%93/Copy%20as%20Go%20struct%20beego.groovy
[sqlx]: https://github.com/shanghairanking/docs/blob/main/3.%E6%95%B0%E6%8D%AE%E5%BA%93/Copy%20as%20Go%20struct%20sqlx.groovy

### JSON 类型

**实体定义**

表字段

```
xxx_info JSON NULL COMMENT 'xxx数据
```

实体字段

```
XxxInfoJSON string   `orm:"xxx_info" json:"-"`
XxxInfo     *XxxInfo `orm:"-"        json:"xxxInfo"`
```

**插入**

> 提要：beego orm 层面操作，能与表中对应的字段类型，必须是 string；但是无论是表字段是 JSON 类型，还是 JSON 的字段是一个对象类型，在 SQL 中都需要体现出其类型

```mysql
INSERT xxx (xxx_info) VALUES
// 例1
(CAST(? AS JSON)),
// 例2
JSON_OBJECT('name', ?, "age", ?)
```

**更新**

```
UPDATE xxx SET xxx_info = JSON_SET(IFNULL(xxx_info, JSON_OBJECT(), '$.name', ?, '$.age', ?) WHERE ...
```

**其他**

注意 JSON 字段的 布尔 类型：如果希望设置一个布尔值而不是，0 或 1，需要在参数占位符后边加上 `IS TRUE` 来表示布尔类型的值

假如 JSON 字段中，存在 Object 类型的字段，那么这个创建的实体是不需要像上面实体定义规则一样定义两个字段（为该实体定义 `orm` 标签本就是无意义的），因为这个实体不是数据表实体，不会直接面都 `bee orm` 的操作层，只有直面的才需要这样处理，才能确保数据能够查询出来

## Git 版本管理

### commit message

```markdown
feat:     增加新功能
change:   功能变动
fix:      修复bug

style:    不影响代码含义的改动，例如去掉空格、改变缩进、增删分号、修改变量名、将魔法值抽取成常量、修改注释等
docs:     只改动了文档相关的内容

refactor: 代码优化、代码重构
perf:     提高性能的改动

build:    go.mod 中类库的版本升级
test:     添加测试或者修改现有测试
```

特殊说明，补交能紧接着遗失的提交，就不需要标签，否则标记为 Fix

### Changelist 

关闭线上的一些严格的校验规则、修改一些框架文件或日志文件等的生成目录配置等的本地开发的非跟踪的文件改动，一般会使用 IntelliJ 的 CVS 提供的 `Changelist` 功能，但是需要注意，假如非提交 `Changelist` 暂存的内容和实际开发需要提交的代码有重叠，提交时就会被忽略！！！

## 技术点

### Graceful

当定义了任何 **异步** 的后台任务机制，都需要考虑在程序结束时的任务防丢

（beego 开启 `Graceful = true` 选项，当请求没有处理完程序是不会结束的）

### 临时文件管理

这里对一些动态生成的文件进行生成位置上的更改说明，如果你也有将所有临时文件、生成文件放在一个统一文件夹中管理的习惯，可以像下边这样做

- **lastupdate.tmp**

  默认位置：项目目录下

  修改：修改 beego 的源码  `%GOPATH%\pkg\mod\github.com\beego\beego\v2@v2.0.1\server\web\parser.go` 中 `lastupdateFilename` 变量的值

  举例：`target/lastupdate.tmp`

- **go_build_项目名.exe**

  默认位置：项目目录下

  修改：修改项目运行配置中的 `Output directory` 项

  举例：`项目所在目录的绝对路径\target`

- **logs**

  配置位置

  ```go
  logs.SetLogger(logs.AdapterFile, fmt.Sprintf(`{"filename":"target/logs/%s.log", "maxdays":15}`
  ```

### orm 日志

在 `Beego 2.0`，中如果希望接口服务实例记录输出所有执行的 SQL 日志，那么需要为 `github.com/beego/beego/v2/client/orm.Debug` 赋值为 `true`

特别说明，`Beego` 在更新至 2.0 后，原先许多重要模块都有发生不同程度的改变，其中原来在 1.0 中使用事务的方式（参照最佳实践），到了 2.0 中行为发生了改变，SQL 日志不再打印

核心代码位置

```
# 重要代码位置
github.com/beego/beego/v2@v2.0.1/client/orm/orm.go:477 (o *ormBase) Raw
github.com/beego/beego/v2@v2.0.1/client/orm/orm_raw.go:86 (o *rawSet) Exec

-- 为什么不打印日志？
因为调用了 Begin 方法后，内部，一个事务标识的 bool 值会变为 true，然后最终会导致下面执行 Exec 的实例类型不同，也就产生了打印日志上的行为差异

-- 非事务
Exec 方法 client/orm.(*dbQueryLog).Exec → ExecContext client/orm/orm_log.go:144 打印日志

-- 开启事务
Exec 方法 client/orm.(*TxDB).Exec → ExecContext client/orm/db_alias.go:250 不打印日志
```

## 最佳实践

### 配置文件

为了业务的发展和需求的迭代，拓展和统一入口，会将一些模块组件的策略放到配置文件中。但是，为了安全，配置文件是不能放到代码仓库中管理的，所以配置文件内容如果有新增一定要注意落实

凡是提供业务数据接口服务的后端，都会需要一个映射核心配置文件的变量实例，来方便获取配置项；几乎所有主流开发语言的项目框架都会提供这样一个机制

Beego 会将从 `conf/app.conf`（注意，配置文件的路径和名字是死的，这也造成后期拓展到 `app.yaml` 遇到阻碍） 中读取到的配置，加载到 `web.AppConfig` → `web.BConfig`

虽然可以通过 `web.AppConfig.Xxx("xxx")` 获取配置文件中的任意配置项，但是没有配置项大纲的提示，总是不合适的，所以，肯定需要自定义配置项到自定义配置实体的映射，因此由项目早期负责人利用 Go 的反射机制，实现了将 `web.AppConfig` 中配置项读取到自定义实体的逻辑（逻辑放在 `init` 函数中），并在没有读取到配置实体对应的配置项时抛出异常，以提示后端开发人员核对配置项

**改进（配置文件格式）**

随着业务的持续发展迭代，项目的配置项可能越来越多，简单的 `ini` 配置形式的配置文件逐渐体现出弊端：可读性差（无法体现出配置的层级关系）、不好维护管理（配置名冗长）

因此，引出 `yaml` 格式的配置文件，使用一些现成的三方包，能够轻松的做到，将 `yaml` 的数据映射到配置实体中；但是问题也来了，Beego 默认的配置文件是写死的，所以只能在主要使用 `yaml` 格式的配置文件基础上，同时保留 Beego 的原生配置 `app.conf`

**修正（配置文件位置）**

> 前置：运行路径的影响

并且在项目中白那些某些 test 的时候，或多或少都会使用工具包下的方法，而自定义配置文件配置数据到配置实体的逻辑就在工具包下，这时，就会因为执行路径的响应，导致无法读取到配置文件报错

因此，应当将配置文件的读取逻辑放置到单独的模块目录，这样使用工具包就不会有各式各样的问题了

### 分页

分页查询的结果一般包含总数，以及当前页的数据，这里存在简单的优化准则，先查询得到数量，如果发现查询得到的数量为 0，则不应该再进行数据库的数据查询

### 线上下路由

因为默认服务端架构都会采用一个网关中间件，网关中间件又需要区分请求是访问前端资源还是后端接口数据，因此，后端数据接口的访问地址会变为：`http://127.0.0.1:8080/v1/userinfo` → `https://xxx.xxx.xxx/api/v1/userinfo`（多了 `api`）

### 加密

目前，前后端约定好了一种对称加密的方法，所有需要涉及重要隐私权限的地方都需要使用到该加密规则

使用需要引入 `github.com/shanghairanking/helper v0.0.2`

注意，和前端对接时，`enc.NewAESCipher(密钥, openSSL)`的 `openSSL` 需要指定为 true，因为前端加密的方法默认是使用 `openSSL` 的

### Dao

#### 基础

**查询一条（QueryRow）**

- 注意：需要为接收结果的实体初始化

  ```
  var m = new(models.M)
  err := r.Raw(q, id).QueryRow(m)
  ...
  ```

- 注意：需要考虑没有查询到的情况

  `beego orm` 查询一条数据（`QueryRow`），如果 没有找到 或 找到多条，都会返回 error

  一个，这样的错误是不能直接返回给前端的，需要处理，可以直接使用 `help.OneRowErr(err, string)` 包装错误后直接返回返回

  还提供了辅助方法：`help.IsOneRow(err error) bool` 和 `help.IsMultiRows(err error) bool`

**查询多条（QueryRows）**

`beego orm` 查询多条数据（`QueryRows`）；结果集容器并不需要初始化；在没有查询到数据时，并不会报错

**判断 MySQL 驱动返回的错误类型**

`1.0 orm（或 2.0 adapter/orm）`：应该用 `err == orm.ErrNoRows`、`orm.ErrMultiRows`，而不是取出错误的内容去比较

`2.0 client/orm`：也有定义同名的错误实例

#### 事务

文档作者，在大半年的开发实践中，经常会使用在方法局部开头声明接下来要使用变量的方式，使得代码结构更为工整，提高代码的可读性，其中最常定义（几乎是必声明）的就是 `error` 类型的变量，于是便将原来的基于一个 `bool` 类型事务提交标识值，改为 `error` 类型；实际调用：

```go
var err error

...

o = orm.NewOrm()
_ = o.Begin()
defer help.HandlerDBTransaction(o, &err)

repo.NewXxxRepo(o).Xxx......
```

#### 场景注意

**查询**

- 所有查询：一定要注意 `deleted_at IS NULL`

- 列表查询：要注意，业务场景下，是否要求排序 `ORDER BY ord_no`

- 列表查询：模糊查询条件，`LIKE ?`，在具体的参数指定 `"%" + xxx + "%"`

  注意，SQL 语法中， 除非模糊查询的条件是数字，否则条件的具体值要加上单引号

**创建**（`created_by`、`updated_by`）

- created_at 和 updated_at 在表字段定义时，设置了默认值

**更新**（`updated_by`）

- updated_at 在表字段定义时，设置了自动更新

- `UPDATE` 更新多个字段值，`SET xxx1 = ?` 后面接第二个字段，拼接的不是 `AND`，而是 `逗号`

- `AND 条件1 OR 条件2`，注意 `条件1 OR 条件2` 是否要用括号括起来

**软删除**（`updated_at`、`deleted_by`、`deleted_at`）

- updated_at 会自动更新，但是 updated_by 不更新

- 带有 `含有唯一约束的字段` 和 `deleted_by、deleted_at 字段` 的数据：

  ```mysql
  UPDATE
  	xxx
  SET
  	name = CONCAT(id, '~', name),
  	phone_num = CONCAT(id, '~', phone_num),
  	email = CONCAT(id, '~', email),
  	deleted_at = ?,
  	deleted_by = ?
  WHERE
  	id = ?
  ```

  

#### 批处理

**批量插入**

```go
func (r *xxxRepo) BatchInsert(userId int64, xxxs []xxx.Xxx) error {
    const fieldSum = n
    one := `(` + help.OrmJoinRepeat(fieldSum) + `)`
    
    // sql
    b := strings.Builder{}
    b.WriteString(`
    INSERT 数据库名.表名
    (字段名1, 字段名2, 字段名3, ..., 字段名n, created_by)
    VALUES `)
    b.WriteString(help.JoinRepeat(one, ",", len(xxxs)))
   
    // 实际参数
    params := make([]interface{}, 0, fieldSum*len(xxxs))
    for _, xxx := range xxxs {
        params = append(params, xxx.Field1, ..., xxx.Fieldn, userId)
    }

    _, err := r.Raw(b.String(), params...).Exec()
    return err
}
```

**批量更新**

```go
# 注意：如果表中有其他非空字段，下面这种方式就不好使了（除非手动填一些值）
# Error 1364: Field 'xxx' doesn't have a default value
func (r *xxxRepo) BatchUpdate(userId int64, xxxs []xxx.Xxx) error {
    const fieldSum = n
    one := `(` + help.OrmJoinRepeat(fieldSum) + `)`
    
    b := strings.Builder{}
    b.WriteString(`
    INSERT 数据库名.表名
    (id, 字段名1, 字段名2, ..., 字段名n-1)
    VALUES `)
    b.WriteString(help.JoinRepeat(one, ",", len(xxxs)))
    b.WriteString(`
    ON DUPLICATE KEY
    UPDATE xxx1 = VALUES(xxx1), ..., xxxn-1 = VALUES(xxxn-1), updated_by = ?`)
   
    params := make([]interface{}, 0, fieldSum*len(xxxs))
    for _, xxx := range xxxs {
        params = append(params, xxx.Id, xxx.Field1, ..., xxx.Fieldn-1)
    }

    _, err := r.Raw(b.String(), params, userId).Exec()
    return err
}
```

```go
func (r *xxx) BatchUpdate(userId int64, xxxs []xxx.Xxx) error {
	const fieldSum = n
	one := `
     UNION ALL SELECT ? AS id, ? AS xxx1, ..., ? AS xxxn-1`

	b := strings.Builder{}
	b.WriteString(`
	WITH args AS (
     SELECT
		? AS id, ? AS xxx1, ..., ? AS xxxn-1`)
	b.WriteString(help.JoinRepeat(one, "", len(xxxs)-1))
	b.WriteString(`
    )
	UPDATE 库名.表名 a JOIN args USING(id)
    SET a.xxx1 = args.xxx1, a.updated_by = ?`)

	params := make([]interface{}, 0, fieldSum*len(xxxs))
	for _, f := range xxxs {
		params = append(params, f.Id, f.Xxx1)
	}

	_, err := r.Raw(b.String(), params, userId).Exec()
	return err
}
```

#### 主库

如果项目中有用到主库的数据，要注意和项目关联较大库的查询表名前边一定要带有库名

目前项目的主库有两个：`univ_ranking_a` 和 `univ_ranking_b`，这两个库是一模一样的，只是说哪个库导入了新的数据，就应该被切换为线上实际使用的，所以是轮番切换使用的；这样做也是避免直接更新线上数据库带来的种种弊端且能实现线上应用的快速切库，和备份数据的效果。

对于应用来说，切换数据库就是修改配置文件中配置的数据库名（默认这些数据库都是存放在一个 MySQL 服务端程序中的），因为查询中没有指定数据库名的表查询，默认就是应用这个数据库，为了实现综上所述，在进行和项目应用本身相关程度比较大的那个数据库的查询的表名前面应该都带上这个数据库的名称，具体表现在 `FROM` 和 `JOIN` 关键字后边的表名

### 错误处理

所有的错误处理都放在 `usercase` 层中，因为这一层是不可能避免处理错误的，就不如统一处理；且在 `dao` 层处理，处理的错误提示太抽象

反：`dao` 方面复用，那些真的从数据库层报出的错误，真的没有必要一个一个去重复处理啊

正：`usercase` 中可以抽取提出 `dao` 层的同名方法进行校验，调用这个方法复用就行了

反：那岂不是所有 `dao` 方法都要在 `usercase` 定义一个同名的错误处理，那导致的额外工作量太大了

正：就实际开发的经验来说，能复用的 `dao` 方法也那就几个查询方法

反：......

