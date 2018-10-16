# MIP Cache 改写规则校验工具

MIP Cache改写规则校验工具主要用于校验Cache改写后是否符合Cache改写规则，同时用于支持例行的改写规则Case以及规则本身的更新检查。

## 依赖与安装

1. git clone 项目
2. 运行`npm i`进行项目依赖安装

## 使用方法

本项目主要维护的是Cache的改写规则，以及对应的集成测试Case。在运行Case是否符合规则时，依赖[mip-validator](https://github.com/mipengine/mip-validator)。因此在使用上比较自由，以下推荐几种常用使用方式。

### 命令行调用

- 全局安装了mip-validator，并且是正确的版本依赖（>=1.6.0），可以直接通过如下方式运行规则：

`mip-validator -c ./mip.rules.json < test.html`

**注：其中test.html为改写后待验证的html**

- 未安装全局mip-validator，可以通过本地node_modules调用命令行运行，调用代码如下：

`./node_modules/.bin/mip-validator -c ./mip.rules.json < test.html`

- 在变更或调试规则与Case时，可以通过`make test`查看运行结果。
