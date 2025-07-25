---
slug: '/docs/cli/gen-dao'
title: '数据规范-gen dao'
sidebar_position: 1
hide_title: true
keywords: [GoFrame,gen dao,数据访问对象,数据转化模型,Go代码生成,配置文件管理,命令行工具,数据库连接,ORM配置,项目工程规范]
description: 'GoFrame框架中gen dao命令的使用方法与参数配置。gen dao命令是生成数据访问对象、数据转化模型及实体数据模型的关键工具，支持通过命令行参数和配置文件进行详细配置，适用于多种数据库类型。通过对命令选项的灵活使用，可以满足不同项目的代码生成需求，确保工程设计规范的落地执行。'
---

`gen dao` 命令是 `CLI` 中最频繁使用、也是框架设计的工程规范能否准确落地的关键命令。该命令用于生成 `dao` 数据访问对象、 `do` 数据转化模型及 `entity` 实例数据模型 `Go` 代码文件。由于该命令的参数、选项较多，我们推荐使用配置文件来管理生成规则。
:::tip
关于框架项目工程规范介绍请查看 [代码分层设计](../../框架设计/工程开发设计/代码分层设计.md) 章节。
:::
## 使用方式

大部分场景下，进入项目根目录执行 `gf gen dao` 即可。以下为命令行帮助信息。以下命令行输出仅供参考：

```text
$ gf gen dao -h
USAGE
    gf gen dao [OPTION]

OPTION
    -p, --path                  directory path for generated files
    -l, --link                  database configuration, the same as the ORM configuration of GoFrame
    -t, --tables                generate models only for given tables, multiple table names separated with ','
    -x, --tablesEx              generate models excluding given tables, multiple table names separated with ','
    -g, --group                 specifying the configuration group name of database for generated ORM instance,
                                it's not necessary and the default value is "default"
    -f, --prefix                add prefix for all table of specified link/database tables
    -r, --removePrefix          remove specified prefix of the table, multiple prefix separated with ','
    -rf, --removeFieldPrefix    remove specified prefix of the field, multiple prefix separated with ','
    -j, --jsonCase              generated json tag case for model struct, cases are as follows:
                                | Case            | Example            |
                                |---------------- |--------------------|
                                | Camel           | AnyKindOfString    |
                                | CamelLower      | anyKindOfString    | default
                                | Snake           | any_kind_of_string |
                                | SnakeScreaming  | ANY_KIND_OF_STRING |
                                | SnakeFirstUpper | rgb_code_md5       |
                                | Kebab           | any-kind-of-string |
                                | KebabScreaming  | ANY-KIND-OF-STRING |
    -i, --importPrefix          custom import prefix for generated go files
    -d, --daoPath               directory path for storing generated dao files under path
    -o, --doPath                directory path for storing generated do files under path
    -e, --entityPath            directory path for storing generated entity files under path
    -t1, --tplDaoIndexPath      template file path for dao index file
    -t2, --tplDaoInternalPath   template file path for dao internal file
    -t3, --tplDaoDoPath         template file path for dao do file
    -t4, --tplDaoEntityPath     template file path for dao entity file
    -s, --stdTime               use time.Time from stdlib instead of gtime.Time for generated time/date fields of tables
    -w, --withTime              add created time for auto produced go files
    -n, --gJsonSupport          use gJsonSupport to use *gjson.Json instead of string for generated json fields of
                                tables
    -v, --overwriteDao          overwrite all dao files both inside/outside internal folder
    -c, --descriptionTag        add comment to description tag for each field
    -k, --noJsonTag             no json tag will be added for each field
    -m, --noModelComment        no model comment will be added for each field
    -a, --clear                 delete all generated go files that do not exist in database
    -y, --typeMapping           custom local type mapping for generated struct attributes relevant to fields of table
    -fm, --fieldMapping         custom local type mapping for generated struct attributes relevant to specific fields of
                                table
    -/--genItems                
    -h, --help                  more information about this command

EXAMPLE
    gf gen dao
    gf gen dao -l "mysql:root:12345678@tcp(127.0.0.1:3306)/test"
    gf gen dao -p ./model -g user-center -t user,user_detail,user_login
    gf gen dao -r user_

CONFIGURATION SUPPORT
    Options are also supported by configuration file.
    It's suggested using configuration file instead of command line arguments making producing.
    The configuration node name is "gfcli.gen.dao", which also supports multiple databases, for example(config.yaml):
    gfcli:
      gen:
        dao:
        - link:     "mysql:root:12345678@tcp(127.0.0.1:3306)/test"
          tables:   "order,products"
          jsonCase: "CamelLower"
        - link:   "mysql:root:12345678@tcp(127.0.0.1:3306)/primary"
          path:   "./my-app"
          prefix: "primary_"
          tables: "user, userDetail"
          typeMapping:
            decimal:
              type:   decimal.Decimal
              import: github.com/shopspring/decimal
            numeric:
              type: string
          fieldMapping:
            table_name.field_name:
              type:   decimal.Decimal
              import: github.com/shopspring/decimal
```
:::tip
如果使用框架推荐的项目工程脚手架，并且系统安装了 `make` 工具，也可以使用 `make dao` 快捷指令。
:::
## 配置示例

文件配置示例：

```yaml title="hack/config.yaml"
gfcli:
  gen:
    dao:
    - link:     "mysql:root:12345678@tcp(127.0.0.1:3306)/test"
      tables:   "order,products"
      jsonCase: "CamelLower"

    - link:   "mysql:root:12345678@tcp(127.0.0.1:3306)/primary"
      path:   "./my-app"
      prefix: "primary_"
      tables: "user, userDetail"

    # sqlite需要自行编译带sqlite驱动的gf，下载库代码后修改路径文件（gf\cmd\gf\internal\cmd\cmd_gen_dao.go）的import包，取消注释即可。sqlite驱动依赖了gcc
    - link: "sqlite:./file.db"
```

## 参数说明

| 名称 | 默认值 | 含义 | 示例 |
| --- | --- | --- | --- |
| `gfcli.gen.dao` |  | `dao` 代码生成配置项，可以有多个配置项构成数组，支持多个数据库生成。不同的数据库可以设置不同的生成规则，例如可以生成到不同的位置或者文件。 | - |
| `link`|  | **必须参数**。分为两部分，第一部分表示你连接的数据库类型 `mysql`, `postgresql` 等, 第二部分就是连接数据库的 `dsn` 信息。具体请参考 [ORM使用配置](../../核心组件/数据库ORM/ORM使用配置/ORM使用配置.md) 章节。 | - |
| `path` | `internal` | 生成 `dao` 和 `model` 文件的存储 **目录** 地址。 | `./app` |
| `group` | `default` | 在数据库配置中的数据库分组名称。只能配置一个名称。数据库在配置文件中的分组名称往往确定之后便不再修改。 | `default`<br />`order`<br />`user` |
| `prefix` |  | 生成数据库对象及文件的前缀，以便区分不同数据库或者不同数据库中的相同表名，防止数据表同名覆盖。 | `order_`<br />`user_` |
| `removePrefix` |  | 删除数据表的指定前缀名称。多个前缀以 `,` 号分隔。 | `gf_` |
| `removeFieldPrefix` |  | 删除字段名称的指定前缀名称。多个前缀以 `,` 号分隔。 | `f_` |
| `tables` |  | 指定当前数据库中需要执行代码生成的数据表。如果为空，表示数据库的所有表都会生成。 | `user, user_detail` |
| `tablesEx` |  | `Tables Excluding`，指定当前数据库中需要排除代码生成的数据表。 | `product, order` |
| `jsonCase` | `CamelLower` | 指定 `model` 中生成的数据实体对象中 `json` 标签名称规则，参数不区分大小写。参数可选为： `Camel`、 `CamelLower`、 `Snake`、 `SnakeScreaming`、 `SnakeFirstUpper`、 `Kebab`、 `KebabScreaming`。具体介绍请参考命名行帮助示例。 | `Snake` |
| `stdTime` | `false` | 当数据表字段类型为时间类型时，代码生成的属性类型使用标准库的 `time.Time` 而不是框架的 `*gtime.Time` 类型。 | `true` |
| `withTime` | `false` | 为每个自动生成的代码文件增加生成时间注释 |  |
| `gJsonSupport` | `false` | 当数据表字段类型为 `JSON` 类型时，代码生成的属性类型使用 `*gjson.Json` 类型。 | `true` |
| `overwriteDao` | `false` | 每次生成 `dao` 代码时是否重新生成覆盖 `dao/internal` 目录外层的文件。注意 `dao/internal` 目录外层的文件可能由开发者自定义扩展了功能，覆盖可能会产生风险。 | `true` |
| `importPrefix` | 通过 `go.mod` 自动检测 | 用于指定生成 `Go` 文件的 `import` 路径前缀。特别是针对于不是在项目根目录下使用 `gen dao` 命令，或者想要将代码文件生成到自定义的其他目录，这个时候配置该参数十分必要。 | `github.com/gogf/gf` |
| `descriptionTag` | `false` | 用于指定是否为数据模型结构体属性增加 `description` 的标签，内容为对应的数据表字段注释。 | `true` |
| `noJsonTag` | `false` | 生成的数据模型中，字段不带有json标签 |  |
| `noModelComment` | `false` | 用于指定是否关闭数据模型结构体属性的注释自动生成，内容为数据表对应字段的注释。 | `true` |
| `clear` | `false` | 自动删除数据库中不存在对应数据表的本地 `dao/do/entity` 代码文件。请谨慎使用该参数！ |  |
| `daoPath` | `dao` | 代码生成的 `DAO` 文件存放目录 |  |
| `doPath` | `model/do` | 代码生成 `DO` 文件存放目录 |  |
| `entityPath` | `model/entity` | 代码生成的 `Entity` 文件存放目录 |  |
| `tplDaoIndexPath` |  | 自定义 `DAO Index` 代码生成模板文件路径，使用该参数请参考源码 |  |
| `tplDaoInternalPath` |  | 自定义 `DAO Internal` 代码生成模板文件路径，使用该参数请参考源码 |  |
| `tplDaoDoPath` |  | 自定义 `DO` 代码生成模板文件路径，使用该参数请参考源码 |  |
| `tplDaoEntityPath` |  | 自定义 `Entity` 代码生成模板文件路径，使用该参数请参考源码 |  |
| `typeMapping` |  | **从版本v2.5开始支持**。用于自定义数据表字段类型到生成的Go文件中对应属性类型映射。 |  |
| `fieldMapping` |   | **从版本v2.8开始支持**。用于自定义数据表具体字段到生成的Go文件中对应属性类型映射。|    | 
| `shardingPattern` |   | **从版本v2.9开始支持**。用于自定义数据表分表规则。|    | 


### 参数：`typeMapping`

参数`typeMapping`支持配置数据库字段类型对应的`Go`数据类型，默认值为：
```yaml
decimal:
  type: float64
money:
  type: float64
numeric:
  type: float64
smallmoney:
  type: float64
```
该配置支持通过`import`配置项引入第三方包，例如：
```yaml
decimal:
  type:   decimal.Decimal
  import: github.com/shopspring/decimal
```

### 参数：`fieldMapping`

参数`fieldMapping`提供细粒度的字段类型映射配置，支持配置指定数据库字段生成的`Go`数据类型。除了配置名称不一样外，配置内容与`typeMapping`一致。配置示例：
```yaml
paid_orders.amount:
  type:   decimal.Decimal
  import: github.com/shopspring/decimal
```
示例中，`paid_orders`为表名称，`amount`为字段名称，`type`表示生成的`Go`代码中对应的数据类型名称，`import`表示生成的代码中需要引入第三方包。

### 参数：`shardingPattern`

参数`shardingPattern`用于配置分库分表规则。该参数在`v2.9`版本中新增，用于识别和处理分片表（`Sharding Tables`）。分片表是指按照某种规则拆分的多个具有相同结构的数据表，例如按照时间分片的`orders_202301`、`orders_202302`等表，或者按照用户ID分片的`users_0001`、`users_0002`等表。

#### 参数说明

- 类型：字符串数组
- 格式：使用`?`作为通配符来匹配表名中的分片部分
- 作用：将多个匹配同一模式的表识别为同一个逻辑表，并生成支持分片的`DAO`代码

#### 工作原理

当使用`shardingPattern`参数时，`gen dao`命令会：

1. 根据提供的模式匹配数据库中的表名
2. 将匹配同一模式的多个表识别为同一个逻辑表
3. 移除表名中的分片标识符部分
4. 为该逻辑表生成支持分片的`DAO`代码

#### 参数示例

假设数据库中有以下表：

```text
users_0001
users_0002
users_0003
products
```

使用以下`shardingPattern`配置：

```yaml
gendao:
  - link: "mysql:root:12345678@tcp(127.0.0.1:3306)/test"
    shardingPattern:
      - "users_?"
```

生成的结果将包含：

1. 一个名为`users.go`的`DAO`文件，而不是为每个分片表生成单独的`DAO`文件
2. 该`DAO`文件中会包含分片配置代码，自动处理对不同分片表的操作
3. 产品表会正常生成`products.go`的`DAO`文件

生成的`DAO`文件示例：
```go
// ...

type usersDao struct {
    *internal.UsersDao
}

var (
    // Users is a globally accessible object for table users_0001 operations.
    Users = usersDao{internal.NewUsersDao(userShardingHandler)}
)

// userShardingHandler is the handler for sharding operations.
// You can fill this sharding handler with your custom implementation.
func userShardingHandler(m *gdb.Model) *gdb.Model {
    m = m.Sharding(gdb.ShardingConfig{
        Table: gdb.ShardingTableConfig{
            Enable: true,
            Prefix: "",
            // Replace Rule field with your custom sharding rule.
            // Or you can use "&gdb.DefaultShardingRule{}" for default sharding rule.
            Rule: nil,
        },
        Schema: gdb.ShardingSchemaConfig{},
    })
    return m
}

// ...
```
其中，您需要自行设定`DAO`文件中的`userShardingHandler`函数中`Sharding`的分表规则，也可以设定分库规则。如果生成分表`DAO`文件但是未手动设置分表规则，直接调用`DAO`对象将会报错。


#### 多个分片模式

可以同时指定多个分片模式的配置示例：

```yaml
gendao:
  - link: "mysql:root:12345678@tcp(127.0.0.1:3306)/test"
    shardingPattern:
      - "users_?"
      - "orders_?"
```

这将同时处理`users_`和`orders_`前缀的分片表，生成`users.go`和`orders.go`的`DAO`文件。

#### 注意事项

1. 分片表必须具有相同的表结构。
2. 分片标识符可以是数字、日期或其他格式，只要能用`?`通配符匹配即可。
3. 生成的`DAO`代码会自动包含所有匹配的分片表，如果后续添加新的分片表，需要重新生成`DAO`代码。
4. 你需要在生成的`DAO`文件中，为每个分片表添加`Sharding`方法，用于指定分表规则，具体请参考章节：[ORM分库分表-分表特性](../../核心组件/数据库ORM/ORM分库分表/ORM分库分表-分表特性.md)。

## 使用示例

仓库地址： [https://github.com/gogf/focus-single](https://github.com/gogf/focus-single)

![](/markdown/a02af38b70bb31224361565570e40789.png)

1、以下 `3` 个目录的文件由 `dao` 命令生成：

| 路径 | 说明 | 详细介绍 |
| --- | --- | --- |
| `/internal/dao` | 数据操作对象 | 通过对象方式访问底层数据源，底层基于 `ORM` 组件实现。往往需要结合 `entity` 和 `do` 共同使用。该目录下的文件开发者可扩展修改。 |
| `/internal/model/do` | 数据转换模型 | 数据转换模型用于业务模型到数据模型的转换，由工具维护，用户不能修改。工具每次生成代码文件将会覆盖该目录。关于 `do` 文件的介绍请参考：<br />- [数据模型与业务模型](../../框架设计/工程开发设计/数据模型与业务模型.md)<br />- [DAO-工程痛点及改进](../../框架设计/工程开发设计/DAO封装设计/DAO-工程痛点及改进.md)<br />- [利用指针属性和do对象实现灵活的修改接口](../../核心组件/数据库ORM/ORM最佳实践/利用指针属性和do对象实现灵活的修改接口.md) |
| `/internal/model/entity` | 数据模型 | 数据模型由工具维护，用户不能修改。工具每次生成代码文件将会覆盖该目录。 |

2、 `model` 中的模型分为两类： **数据模型** 和 **业务模型**。

**数据模型：** 通过 `CLI` 工具自动生成 `model/entity` 目录文件，数据库的数据表都会生成到该目录下，这个目录下的文件对应的模型为数据模型。数据模型即与数据表一一对应的数据结构，开发者往往不需要去修改并且也不应该去修改，数据模型只有在数据表结构变更时通过 `CLI` 工具自动更新。数据模型由 `CLI` 工具生成及统一维护。

**业务模型：** 业务模型即是与业务相关的数据结构，按需定义，例如 `service` 的输入输出数据结构定义、内部的一些数据结构定义等。业务模型由开发者根据业务需要自行定义维护，定义到 `model` 目录下。

3、 `dao` 中的文件按照数据表名称进行命名，一个数据表一个文件及其一个对应的 `DAO` 对象。操作数据表即是通过 `DAO` 对象以及相关操作方法实现。 `dao` 操作采用规范化设计，必须传递 `ctx` 参数，并在生成的代码中必须通过 `Ctx` 或者 `Transaction` 方法创建对象来链式操作数据表。

![](/markdown/f0da330685c6cfd82ba1c0254dfdbe39.png)

## 注意事项

### 需要手动编译的数据库类型

`gen dao` 命令涉及到数据访问相关代码生成时，默认支持常用的若干类型数据库。如果需要 `Oracle` 数据库类型支持，需要开发者自己修改源码文件后自行本地手动编译生成 `CLI` 工具随后安装，因为这两个数据库的驱动需要 `CGO` 支持，无法预编译生成给大家直接使用。

![](/markdown/7f849959c13d224393b93d6b371e8ae0.png)

### 关于 `bool` 类型对应的数据表字段

由于大部分数据库类型都没有 `bool` 类型的数据表字段类型，
我们推荐使用`bit(1)`来表示字段的`bool`类型，而非`tinyint(1)`或者`int(1)`。因为`tinyint(1)/int(1)`字段类型表示的范围是`-127~127`，通常可能会被用作状态字段类型。而`bit(1)`的类型范围为`0/1`，可以很好的表示`bool`类型的两个值`false/true`。

例如，表字段：

![](/markdown/50992d00a792555d2946d294975e9ec4.png)

生成的属性：

![](/markdown/4bb766d64e607a33c1a6fbf20c742924.png)
