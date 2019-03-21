# Bihu-2018

starlight 项目的主要后台API系统

采用微服务架构设计，目前的服务如下：

* [keystone](keystone/README.md) 核心服务，为提起服务提供基础组件
* [auth](auth/README.md) 认证服务，提供账号的管理和认证
* [account](account/README.md) 用户组（即租户）的管理
* [app](app/README.md) 应用管理
* hpcres HPC 资源调用
* job 系统作业管理
* stardust 星光积分管理
* ticket 用户工单系统
* yubaaba 用户机时管理系统

部分服务目前【应】只提供内部／管理员调用, 见内部说明。 

# Quick Start 

星光的大部分接口在使用时需要使用 Bihu-Token 作为令牌进行请求，令牌默认有效期为24小时， 这里演示通过curl 请求获取Bihu-Token 并通过 token 提交应用 lammps 的例子

1. 获取Bihu-Token

```
$ curl -X POST   https://starlight.nscc-gz.cn/api/keystone/token   -H 'Content-Type: application/json' -d '{"spec":{"username":"'${username}'","password":"'${password}'"}}'
{"apiVersion":"v0.0.3","kind":"keystone.token","spec":{"username":"####","token":"eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.####.####","tokenKind":"Bihu-Token"}}
```

2. 提交作业

```
$ curl -X POST   https://starlight.nscc-gz.cn/api/app/exec/jupyterhpc/0   -H 'Content-Type: application/json' -H "Bihu-Token: ${Bihu_Token}" -d '{"jupyter_genpassword": "true","jupyter_password": "testpasword",}'

```
