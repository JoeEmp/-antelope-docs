### 步骤

在`modules/step/__init__.py`下有步骤名和步骤实现类的映射关系的枚举,框架在运行时会通过这个枚举转化成相应的类

```python
@unique
class STEP_TYPE(Enum):
    # 支持的步骤 断言为驼峰命名，其他步骤名字为下划线
    sql_exec = SqlExecStep
    assertEqual = AssertEqualStep
    set_step = SetStep
    assertNotEqual = AssertNotEqualStep
    case = CaseStep
    func = MyFunctionStep
    request = RequestStep
    assertIn = AssertInStep
    assertJsonSchema = AssertJsonInStep
    judge = JudgeStep
    if_step = IfStep
    else_step = ElseStep
    elif_step = ElifStep
    assertLenEqual = AssertLenEqualStep
    assertLenNotEqual = AssertLenNotEqualStep
    echo = EchoStep
    sleep = SleepStep
    assertListEqual = AssertListEqualStep
    assertJson = AssertJsonStep
    assertNotIn = AssertNotInStep
    assertValueType = AssertValueType
    template = TemplateBlockStep
    case_v2 = CaseStepV2
    set_by_json = SetByJsonStep
```

### sql 步骤

sql 步骤，可指定结果并把结果设置到用例变量中。sql 步骤是一个特殊步骤，db 的链接是由全局变量提供，如果选中环境下没有 db 配置则会报错。

| 参数            | 是否必填 | 类型 | 描述                                         |
| --------------- | -------- | ---- | -------------------------------------------- |
| db              | 必填     | str  | 变量的名字，如果已经变量名已经存在，则会覆盖 |
| sql             | 必填     | any  | 可以是任意值，也可以是已经存在的变量         |
| sql_result_name | 必填     | str  | 查询结果存放的名字                           |

全局变量

```yaml
demo:
  db:
    regression_test:
      filename: regression_test.db
      type: sqlite
```

步骤

```yaml
- sql_exec:
    db: regression_test
    sql: |
      INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY) 
      VALUES (1, 'Paul', 32, 'California', 20000.00 )
    sql_result_name: row
- sql_exec:
    db: regression_test
    sql: select * from COMPANY where ID = 1
    sql_result_name: row
```

#### requests 步骤

请求步骤是一个特殊的步骤，用例模板的 case 模块处于同一层级。

| 参数                | 是否必填 | 类型            | 描述                   |
| ------------------- | -------- | --------------- | ---------------------- |
| method              | 必填     | str             | http 的请求方式        |
| host                | 必填     | str             | 域名                   |
| url                 | 必填     | str             | 除去域名的路径         |
| header              | 选填     | json 格式的 str | 请求头                 |
| body                | 选填     | json 格式的 str | 请求体                 |
| response_value_name | 选填     | str             | 响应存到用例变量的名字 |

使用例子

```yaml
request:
  method: "POST"
  host: "https://test.dianyoumi.com"
  url: "/api/admin/base/open/login"
  headers: {"tenantId":"1004200"}
  body: {"phone":"{phone}","password":"xxx","captchaId":"cd3be030-960c-11ec-87db-1f123c911c33","verifyCode":"7647","tenantId":"1004200"}
  response_value_name: response

case:
	# 该行是执行，不使用该行则不会请求
  - request: request
```

#### 设置变量步骤

这些步骤可以让我们设定指定值到局部变量里面

#### set_step

| 参数  | 是否必填 | 类型 | 描述                                         |
| ----- | -------- | ---- | -------------------------------------------- |
| name  | 必填     | str  | 变量的名字，如果已经变量名已经存在，则会覆盖 |
| value | 必填     | any  | 可以是任意值，也可以是已经存在的变量         |

set_by_json
| 参数 | 是否必填 | 类型 | 描述 |
| ----- | -------- | ---- | -------------------------------------------- |
| - | 必填 | obj |yaml 的 json 对象|

使用例子

```yaml
# 局部变量 {"username":"Joe","data":{"phone":"135-"}}
- set_step:
    name: "address"
    value: "海珠区"
# 局部变量 {"username":"Joe","data":{"phone":"135-"},"address":"海珠区"}
- set_step:
    name: "call"
    value: "${data}.phone"
# 局部变量 {"username":"Joe","data":{"phone":"135-"},"address":"海珠区","call":"135-"}
- set_by_json: { "frist_name": "koko", "last_name": "JoJo" }
# 局部变量 {"username":"Joe","data":{"phone":"135-"},
#         "address":"海珠区","call":"135-",
#         "frist_name": "koko", "last_name": "JoJo"}
```

#### 分支步骤

##### if、elif、else

分支步骤是一个特别的步骤，该步骤下的第一个步骤为判断条件只能是 judge 步骤作为条件判断。**judge 结果为真**则执行下方步骤，**judge 结果为假则跳出不执行。整个是功能设计同 python**。

```yaml
- set_step:
    name: code
    value: 1
- if:
    - judge: "${code} == 1"
    ... 若干步骤
- elif:
    - judge: "${code} == 0"
    ... 若干步骤
-else:
    ... 若干步骤
```

#### 模版步骤

##### template

它类似于我们编程中的函数,它可以把一些步骤封装好,来增强我们的复用性，`template`和`request`一致，都是在用例的同一层级定义，并且必须以`template`为开头才能被正常引用。

```yaml
request: ....

template_normal_repsonse:
  - assertEqual:
      - "${repsonse}.code"
      - 200
      - 返回异常，返回应为失败

template_error_repsonse:
  - assertEqual:
      - "${repsonse}.code"
      - 500
      - 返回异常，返回应为失败

case1:
  use_value: normal
  test_step:
    - request: requset
    - template: template_normal_repsonse

case_errror1:
  use_value: error1
  test_step:
    - request: requset
    - template: template_error_repsonse

case_errror2:
  use_value: error2
  test_step:
    - request: requset
    - template: template_error_repsonse
```

同时我们支持跨用例引用模版，方便我们共用一些步骤，我们以上面的例子拆分

结构目录如下

```bash
# shop_case
#   - common
#       response_template.yaml
#   - business
#       - case1
#           - case1_template.yaml
#       - case2
#           - case2_template.yaml
```

```yaml
# response_template.yaml
_version: 2.0

template_normal_repsonse:
  - assertEqual:
      - "${repsonse}.code"
      - 200
      - 返回异常，返回应为失败

template_error_repsonse:
  - assertEqual:
      - "${repsonse}.code"
      - 500
      - 返回异常，返回应为失败
```

```yaml
# case1_template.yaml
_version: 2.0

request: ....

case1:
  use_value: normal
  test_step:
    - request: requset
    # 引用 common/response_template.yaml文件的template_normal_repsonse模版
    - template: common.response_template.template_normal_repsonse
```

#### 断言步骤

##### assertEqual

参数为数组，前两个元素用于比较，第三个元素为错误信息<br />使用例子

```yaml
- assertEqual:
    - 1000
    - "{code}"
    - 登录失败
```

###### assertNotEqual(assertEqual)

使用例子

```yaml
- assertNotEqual:
    - 1000
    - "{code}"
    - 登录失败
```

###### assertIn

参数可以为数组，也可以是关键字。如果是关键字内容

```yaml
- assertIn:
    - "a"
    - "abc"
    - 设备名错误

- assertIn:
    member: "d"
    container: "abc"
    msg: 设备名错误
```

###### assertNotIn(assertIn)

```yaml
- assertNotIn:
    - "a"
    - "abc"
    - 设备名错误

- assertNotIn:
    member: "d"
    container: "abc"
    msg: 设备名错误
```

###### assertJson

判断预期结果(Json)和实际 json 一致

```yaml
- assertListEqual:
	- "{sql_select}"      # 预期结果
  - "{devices}"         # 实际结果
  - "{error_msg}"       # 错误信息
  - "{is_ignore_null}"  # 可选,默认不忽略null值的key 使用了item为json的步骤
```

###### assertLenEqualStep

判断变量为指定长度

```yaml
- assertLenEqual:
    obj: "{sql_select}"
    len: 3
    msg: 设备配置不等于3，请检查设备初始化
```

###### assertLenNotEqualStep

判断变量不为指定长度

```yaml
- assertLenNotEqual:
    obj: "{sql_select}"
    len: 3
    msg: 设备配置不等于3，请检查设备初始化
```

##### 自定义方法步骤

该步骤会使用 funciton 文件下用户自定义的 python 方法。方法的定义中需要定义 case_value 以便使用用例变量，如果你不需要使用 case_value，则可以定义\*\*kwargs 来忽略该参数

| 参数        | 是否必填 | 类型 | 描述                                                               |
| ----------- | -------- | ---- | ------------------------------------------------------------------ |
| name        | 是       | str  | 完整的引用路径                                                     |
| args        | 否       | list | 参数，按顺序入参                                                   |
| kwargs      | 否       | dict | 如果方法无参数，则是选填，内容为参数，会入参到指定方法对应的定义中 |
| func_result | 否       | str  | 返回结果存入用例变量文件名                                         |

使用例子

```yaml
# 假设有这样的配置
# runner.py
# - case
# - function
#   joe_func.py
			# def echo_case_value1(**kwargs):
			# def echo_case_value2(case_value,**kwargs):
      # def echo_name(someone,**kwargs)


- func:
    name: "function.joe_func.echo_name"
    kwargs:
      someone: "{name}"

- func:
    name: "function.joe_func.echo_name"
    args:
     	- "{name}"
# case_value 不需要指定入参
- func:
    name: "function.joe_func.echo_case_value1"

- func:
    name: "function.joe_func.echo_case_value2"
```

#### sleep(思考时间)

| 参数 | 是否必填 | 类型   | 描述 |
| ---- | -------- | ------ | ---- |
| -    | 必填     | number | 秒   |

```yaml
- sleep: 1.1
```

#### echo(思考时间)

打印变量到日志上，会打印变量类型和变量内容

| 参数 | 是否必填 | 类型 | 描述 |
| ---- | -------- | ---- | ---- |
| -    | 必填     | Any  | 对象 |

```yaml
- ehco: ${name}
- ehco: 1.2
- ehco: 1
- ehco: "Molody"
- ehco: {}
- ehco: []
```
