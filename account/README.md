# account 服务

 提供租户相关信息的服务

# models

见 [account/models.go](./models.go)

## APIs

### /account restful APIs

* - [x] `POST /account [admin]`

* - [x] `GET /account [admin]`

* - [x] `GET /account [staff]`

* - [x] `GET /account/<accountId:int>`

* - [x] `GET /account/<accountName:string>`

* - [x] `PATCH /account/<accountId:int>`

* - [x] `POST /account/sync [admin]` 从接口 hpcapi/cpuhours 获取全部租户信息

### /approval restful APIs 用户申请表的信息

* - [x] `POST /approval [admin]`

* - [x] `GET /account [admin]`

* - [x] `GET /account [staff]`

* - [x] `GET /approval`

* - [x] `GET /approval/<accountId:int>`

* - [x] `GET /approval/<accountName:string>`

* - [x] `PATCH /approval/<accountId:int>`

