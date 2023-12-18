# 快速开始(执行一个简单用例)

`python3 runner demo_case/aiot_login_min`

这里运行了一个简单的登录接口。
我们先填写一个简单的请求模板，包含请求方式`method`、主机名`host`、url、请求头`headers`(可选)、请求体`body`(可选)、响应保存的变量名
然后我们在模块下写步骤。

```yaml
# 请求模版
request:
  method: "POST"
  host: "http://localhost"
  url: "login"
  headers: {"from":"antelope"}
  body: {"username":"amdin","password":"123456"}
  response_value_name: response

# case模块存在步骤里
case:
  test_step:
    # 发起请求
    - request: request
  ...

```

### 使用项目配置

由于各个配置的不同，框架提供配置初始化的脚手架来方便配置环境

```bash
python3 init_project_conf.py ipc {project_name}
```

假设{project_name}为 shop，我们会生成 shop_case 的文件夹，里面包含邮件配置文件，数据库配置文件，框架运行时也会引用这两个文件，而不是一级目录下的两个配置文件。同时 cli.py 生成的用例模版也会默认建在 shop_case 的文件夹下。
如果你不需要使用生成的文件，则输入 n 拒绝即可。

![](image/init_project.gif)

### 测试回归

在测试失败时，框架会运行生成一个 error_suite.yaml 文件，并打印相应的命令行给我们，我们直接运行即可回归测试。

yaml 模板语法
框架使用 yaml 作为用户的输入规范，来减低 ast 的开发工作量
一篇文章掌握 yaml
框架的语法见功能设计
使用变量执行用例
变量初始化有会有 3 个阶段
初始化全局变量 -> 初始化用例环境变量 -> 初始化用例变量
使用调试模式执行用例
使用调试模式只要在末尾添加-debug 即可，调试模式下，将打印 debug 等级日志。如果无法排查一些使用的问题，使用-debug 执行将转给开发者，有助于排查。

### 使用优先级控制执行用例

优先级和我们传统的测试用是一致的。旨在在短时间内执行重要的测试用例，以期达到在相对有限的时间内对产品有最大的质量保障。

### 快速导入

提供 xlxs_to_case.py 文件
