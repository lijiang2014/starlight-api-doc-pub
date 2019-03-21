# keystone 服务模块

keystone 模块提供一些基础的，重要的，可能会被其他业务服务调用的功能 

## SubSystemService `/subsystems`

显示有哪些子系统；目前并未启用

## ResourceService `/resource`

显示资源信息；目前并未启用，相关逻辑功能被 `catalog` 模块代替

## CAService `/ca`

为其他服务提供 CA 文件系统，容器云有调用。实际可以用 kubernetes 的 ConfigMap 代替

主要接口：

* `GET /ca?secure=<secure-token>` 通过secure 来获取 CA 文件的内容

## GroupService '/groups' 

提供用户组的管理，此组非租户组，是星光意义上的组，目前比较重要的应用场景有二 ：

* 区分不同部门的员工 Staff ， 进而控制某些管理功能的访问权限

* 跨租户对用户分类，限制某些资源的使用，如 特定的 App

接口需要同步更新 `group`, `user_groups` 两个表，

主要接口：

* 通用RESTFUL接口:
  * `GET /groups` 
  * `GET /groups/<Id>` 
  * `POST /groups` 
  * `DELETE /groups/<Id>`
  * `PATCH /groups/<Id>` 暂时不推荐使用，请通过 addusers／rmusers 来对 用户表进行操作，否则可能会出错 （TO-DO #1 ：完善这部分逻辑）

* `GET /groups/<GroupName>` 通过GroupName 来进行检索 
* `POST /groups/addusers` 单独的批量添加用户到特定组的接口
* `POST /groups/rmusers`  单独批量删除用户到特定组的接口

## PrivilegeService '/rbac' 

设置资源的访问控制，即 RBAC 对象， 通过 资源-（操作）-角色-组-用户 的 多重访问控制来进行资源的细粒度可控访问控制

操作对象：

* [`comm.RBAC`](../comm/rbac.go)  
* [`RBACGroupMember`](./models.go) 辅助查询使用，无需直接操作 

主要接口：

* 通用RESTFUL接口:
  * `GET /rbac` 
  * `GET /rbac/<Id>` 
  * `POST /rbac` 
  * `DELETE /rbac/<Id>`
  * `PATCH /rbac/<Id>`

* `GET /rbac/<RUUID>` 通过指定的资源对象名称来进行检索 
* `GET /rbac?search_by_group=<groupName>`  查询对特定group 进行的访问控制设置

## TokenService `/token`

复杂Token 的登录接口，用来为用户生成包含完整信息的 Token 


主要接口：

* `POST /token` 直接登录来获取复杂Token

request example(body):

```
{
	"spec": {
		"username": "{{USERNAME_USER}}",
		"password": "{{PASSWORD_USER}}"
	}
}
```

response example(body):

```
{
    "apiVersion": "v0.0.3",
    "kind": "keystone.token",
    "spec": {
        "username": "sysu_hpcedu_260",
        "token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjozMywiYWNjb3VudCI6InN5c3VfaHBjZWR1IiwidXNlcnMiOnsiMCI6eyJ1aWQiOjk1LCJrZXkiOiJzeXN1X2hwY2VkdV8yNjAiLCJkYXRhIjp7ImVtYWlsIjoiamlhbmcubGlAbnNjYy1nei5uZXQifX0sIjEwMDMiOnsidWlkIjo2MzI4LCJnaWQiOjU4ODksImtleSI6InN5c3VfaHBjZWR1XzI2MCIsImRhdGEiOnsiaG9tZSI6Ii9CSUdEQVRBMS9zeXN1X2hwY2VkdV8yNjAiLCJwYXNzd29yZCI6IntTU0hBfStSZ1hkc1g0MU1WeUVBYlY4dTlIRmFhb0graDdHeC9SIn19LCIxMDQiOnsidWlkIjo2MzI4LCJnaWQiOjU4ODksImtleSI6InN5c3VfaHBjZWR1XzI2MCIsImRhdGEiOnsiaG9tZSI6Ii9CSUdEQVRBMS9zeXN1X2hwY2VkdV8yNjAifX0sIjEyIjp7ImtleSI6InN5c3UtaHBjQGVkdS5jb20ifSwiMTMiOnsidWlkIjo2MzI4LCJnaWQiOjU4ODksImtleSI6InN5c3VfaHBjZWR1XzI2MCIsImRhdGEiOnsiZ3JvdXBuYW1lIjoic3lzdV9ocGNlZHUiLCJob21lIjoiL0JJR0RBVEExL3N5c3VfaHBjZWR1XzI2MCJ9fX0sImV4cCI6MTU1MzE2MDQyMywiaXNzIjoia2V5c3RvbmUudG9rZW4iLCJzdWIiOiJzeXN1X2hwY2VkdV8yNjAifQ.{{HASHED PAYLOAD}}",
        "tokenKind": "Bihu-Token"
    }
}
```

* `GET /token` 通过 简单Token 来获取复杂Token 

# TO-DO 

1. 见 TODO #1
