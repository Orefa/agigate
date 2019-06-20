## 介绍
基于[Caddy](https://github.com/mholt/caddy),使用[Casbin](https://github.com/casbin/casbin)编写的API网关

集成的插件有[forwardproxy](https://github.com/caddyserver/forwardproxy)和[filebrowser](https://github.com/filebrowser/filebrowser)

其中filebrowser经过调整适配了IE11

功能上支持代理、反向代理、文件服务器、API网关

## 文档
1. Caddy文档，地址: https://caddyserver.com/docs
2. forwardproxy文档, 地址: https://caddyserver.com/docs/http.forwardproxy
3. filebrowser文档已经从Caddy文档中移除，使用方式如下：
```
filebrowser /blog D:/ {
  database D:/blog.db
}
```

## API网关说明
### 使用
```
authz {
	loginURL "loginurl1, loginurl2"
	casbinPolicyURL casbinPoliyUrl
	cookieName apigateway
	logoutURL logoutURL
	
	driverName sqlite3
	dataSourceName "./authz.db"
	
	#driverName mysql
	#dataSourceName usr:pwd@tcp(localhost:3306)/casbin?charset=utf8
	
	#driverName mssql
	#dataSourceName "server=localhost;port=1433;user id=usr;password=pwd;database=casbin"
	
	#driverName postgres
	#dataSourceName postgres://usr:pwd@localhost:5432/casbin?sslmode=disable
	
	model authz_model.conf
	whitelist whitelist.csv
	authWhitelist authWhitelist.csv
	headKeyTransform headerKeyTransform.json
}
```
1. `loginURL`和`logoutURL`可支持多URL，多URL时，英文逗号分隔，双引号引起来
2. `cookieName` 可指定，也可不填
3. `casbinPolicyURL` 为获取策略文件信息url
4. `model` 为RBAC策略模型
5. `whitelist`为白名单
6. `authWhitelist`为鉴权白名单
`headKeyTransform`为登录返回值与登录后每次请求后Header携带的Key映射
```json
{
	"userId": "UID",
	"userName": "USRNAME",
	"account": "ACCOUNT",
	"loginTime": "LOGINTIME",
	"rsv": "RSV"
}
```
其中`userId`代表登录返回信息，在登录后请求中Header中对应的Key为`UID`,其值相同;其它依次类推

其中白名单`whitelist`鉴权白名单`authWhitelist`对应的文件内容格式为：

每行一个url，格式method1,method2,...:url或url，如：
```
get,post:url1
/blog/*
```

### 登录URL返回格式要求如下
----------------------
```go
type LoginResponse struct {
	Flag      bool      `json:"flag"`
	Message   string    `json:"message"`
	ErrorCode string    `json:"errorCode"`
	Result    *UserInfo `json:"result"`
}

// UserInfo login user info
type UserInfo struct {
	UserID            string `json:"userId"`
	Account           string `json:"account"`
	Username          string `json:"userName"`
	Rsv               bool   `json:"rsv,omitempty"`
	SessionTimeout    int    `json:"sessionTimeout"`
	SessionNumPerUser int64  `json:"sessionNumPerUser"`
	LoginTime         int64  `json:"loginTime"`
}
```
返回值json中`rsv`代表是否为内置超级用户

### casbinPoliyUrl要求返回格式如下：
---------------------------------

```go
type UserPerm struct {
	Account        string                   `json:"account"`
	Rsv            bool                     `json:"rsv"`
	RoleCodes      []string                 `json:"roleCodes"`
	RoleCodeURLMap map[string]*MethodURLMap `json:"roleCodeUrlsMap"`
}

type MethodURLMap struct {
	Get     []string `json:"get"`
	Head    []string `json:"head"`
	Post    []string `json:"post"`
	Put     []string `json:"put"`
	Patch   []string `json:"patch"`
	Delete  []string `json:"delete"`
	Options []string `json:"options"`
	Trace   []string `json:"trace"`
	All     []string `json:"all"`
}
```
