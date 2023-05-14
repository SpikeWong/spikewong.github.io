---
id: 446
title: 'Golang: beego crud 实践'
date: '2020-05-13T22:52:21+08:00'
author: Spike
layout: post
guid: 'http://box5781.temp.domains/~spikeooo/?p=446'
permalink: /2020/05/13/golang-beego-crud-%e5%ae%9e%e8%b7%b5/
post_views_count:
    - '1222'
image: /wp-content/uploads/2019/05/Golang-150x150.png
categories:
    - Golang
---

## 概览

本文将使用 beego 框架实现一个 crud 的 api。通过 beego 的脚手架 bee 工具我们将快速地完成项目地初始化和生成相关配置文件。那么现在就开始吧！

## 快速上手

#### 项目环境配置

请先确保你已安装 beego 和 bee,如果还未安装，请按照 [beego 安装指南](https://beego.me/docs/install/) 和 [bee 安装指南](https://beego.me/docs/install/bee.md)。

#### 生成你的项目

```bash
bee api api-template
```

上述命令将在 $GOPATH/src 生成一个样例程序，其中包括 object 和 user 相关的 controller 与 model, 该目录下的文件结构如下所示

```
|____routers
| |____router.go
|____tests
| |____default_test.go
|____models
| |____object.go
| |____user.go
|____controllers
| |____object.go
| |____user.go
|____conf
| |____app.conf
|____main.go
```

当我们在使用命令生成 api 项目的时候指定数据库连接 conn 时，bee 就能够根据你的数据库表结构生成模型和 controller。虽然我在 GOPATH 目录下使用 bee 命令，并且指定了 conn ，但是一直在出现下面的错误。

```
spike:src apple1$ bee api practice-api -conn="root:password@tcp(127.0.0.1:3306)/beego_blog"
______
| ___ \
| |_/ /  ___   ___
| ___ \ / _ \ / _ \
| |_/ /|  __/|  __/
\____/  \___| \___| v1.10.0
2020/05/13 21:35:04 INFO     ▶ 0001 Creating API...
    create   /usr/local/opt/go/src/practice-api
    create   /usr/local/opt/go/src/practice-api/conf
    create   /usr/local/opt/go/src/practice-api/controllers
    create   /usr/local/opt/go/src/practice-api/tests
    create   /usr/local/opt/go/src/practice-api/conf/app.conf
    create   /usr/local/opt/go/src/practice-api/main.go
2020/05/13 21:35:04 INFO     ▶ 0002 Using 'mysql' as 'driver'
2020/05/13 21:35:04 INFO     ▶ 0003 Using 'root:password@tcp(127.0.0.1:3306)/beego_blog' as 'conn'
2020/05/13 21:35:04 INFO     ▶ 0004 Using '' as 'tables'
2020/05/13 21:35:04 INFO     ▶ 0005 Analyzing database tables...
2020/05/13 21:35:04 FATAL    ▶ 0006 Cannot generate application code outside of GOPATH '/usr/local/opt/go' compare with CWD '/usr/local/opt/go/src/practice-api'
```

最终 bee 生成的项目目录如下所示

```
.
|____routers
|____tests
|____models
|____controllers
|____conf
| |____app.conf
|____main.go
```

#### 项目配置

打开 conf/app.conf 文件

```
appname = my-api
httpport = 8080
httpaddr = 127.0.0.1
runmode = dev
# api 项目不需要模板渲染模板
autorender = false
# http 请求时，允许返回原始请求体数据字节
copyrequestbody = true
# 开启自动化生成文档
EnableDocs = true

db_driver_name = mysql
sqlconn = root:password@/beego_template?charset=utf8&loc=Asia%2FShanghai
```

#### 模型设计

bee 生成 migration 文件

```
bee generate migration user
```

database/migrations/20200429\_231223\_users.go

```
package main

import (
    "github.com/astaxie/beego/migration"
)

// DO NOT MODIFY
type Users_20200429_231223 struct {
    migration.Migration
}

// DO NOT MODIFY
func init() {
    m := &Users_20200429_231223{}
    m.Created = "20200429_231223"

    migration.Register("Users_20200429_231223", m)
}

// Run the migrations
func (m *Users_20200429_231223) Up() {
    m.SQL("CREATE TABLE users(id INT AUTO_INCREMENT PRIMARY KEY, username VARCHAR(255), password VARCHAR(255), email VARCHAR(255))")
}

// Reverse the migrations
func (m *Users_20200429_231223) Down() {
    m.SQL("DROP TABLE users")
}

```

在 models 目录下新建 models.go 和 user.go

models/models.go

```
package models

import (
    "github.com/astaxie/beego/orm"
    "fmt"
    "github.com/astaxie/beego"
    _ "github.com/go-sql-driver/mysql"

)

type User struct {
    Id int `json:"id"`
    Name string `json:"name"`
    Password string `json:"password"`
    Email string `json:"email"`
}

func (*User) TableEngine() string {
    return engine()
}

func (*User) TableName() string {
    return "user"
}

func engine() string {
    return "INNODB DEFAULT CHARSET=utf8 COLLATE=utf8_general_ci"
}

func init() {
    driverName := beego.AppConfig.String("db_driver_name")
    sqlconn := beego.AppConfig.String("sqlconn")
    err := orm.RegisterDataBase("default", driverName, sqlconn, 30, 200)
    if err != nil {
        fmt.Println("连接数据库出错, %s", err.Error())
        return
    }
    fmt.Println("成功连接数据库")

    orm.RegisterModel(new(User))
}

```

models/user.go

```
package models

import (
    "github.com/astaxie/beego/orm"
    "my-api/utils"
    "strconv"
)

func InsertUser(user *User)(int64, error){
    orm := orm.NewOrm() 
    return orm.Insert(user)
}

func GetUser(uid int)(u *User, err error){
    orm := orm.NewOrm()
    user := &User{ Id: uid }
    err = orm.Read(user)

    return user, err
}

func DeleteUser(uid int)(b bool, err error) {
    orm := orm.NewOrm()
    user := &User{ Id: uid }
    _, err = orm.Delete(user)

    if err != nil {
        return false, err
    }

    return true, err
}

func GetAllUsers(p int, size int) (u utils.Pagination, err error) {
    o := orm.NewOrm()
    user := new(User)
    var users []User
    qs := o.QueryTable(user)
    count, _ := qs.Count()
    _, err = qs.Limit(size).Offset((p - 1) * size).All(&users)
    c, _ := strconv.Atoi(strconv.FormatInt(count, 10))
    return utils.Paginate(c, p, size, users), err
}
```

#### 接口设计

error.go 对统一对错误情况进行处理  
controllers/error.go

```
package controllers

import (
    "github.com/astaxie/beego"
)

type ErrorController struct {
    beego.Controller
}

func (c *ErrorController) Error404() {
    c.Data["content"] = "page not found"
}

func (c *ErrorController) Error500() {
    c.Data["content"] = "server error"
}
```

上面的实践并不怎么好，错误处理逻辑隐藏掉了错误信息，在生产环境我们可以这么干，但是在测试和开发环境这使我们很难调试！你可以改进一下错误处理逻辑返回详细的报错信息。

controllers/user.go

```
package controllers

import (
    "my-api/models"
    "github.com/astaxie/beego"
    "encoding/json"
    "my-api/utils"
    "strconv"
)

type UserController struct {
    beego.Controller
}

// @Title CreateUser
// @Description create user
// @Param body body models.User true "content for user"
// @Success 200 {string} ok
// @Failure 403 body is empty
// @router / [post]
func (this *UserController) Post() {
    var user models.User
    err := json.Unmarshal(this.Ctx.Input.RequestBody, &user)
    if err != nil {
      this.Abort("500")
    }
    models.InsertUser(&user)
    this.Data["json"] = "ok"
    this.ServeJSON()
}

// @Title GetUser
// @Description get user
// @Param uid path string true "user's id you want to get"
// @Success 200 {object} models.User
// @Failure 403 uid is empty
// @router /:uid [get]
func (this *UserController) Get() {
    uid, _ := this.GetInt(":uid")
    user, err := models.GetUser(uid)
    if err != nil {
        this.Data["json"] = user
     } else {
        this.Abort("500")
     }
     this.ServeJSON()
}

// @Title DeleteUser
// @Description delete user by id
// @Param uid path string true "user's id you want to delete"
// @Success 200 {string} ok
// @Failure 403 uid is empty
// @router /uid [delete]
func (this *UserController) Delete() {
    uid, _ := this.GetInt(":uid")
    _, err := models.DeleteUser(uid)
    if err != nil {
        this.Abort("500")
    }
    this.Data["json"] = "ok"
    this.ServeJSON()
}

// @Title GetAllUsers
// @Description get all users with pagination
// @Param page path string true "the page you want get get"
// @Success 200 {object} models.User
// @Failure 403 uid is empty
// @router /
func (this *UserController) GetAll() {
  currentPage, _ := strconv.Atoi(this.Ctx.Input.Query("page"))
  if currentPage == 0 {
      currentPage = 1
  }
  d, err := models.GetAllUsers(currentPage, utils.PageSize)
  if err != nil {
      this.Abort("500")
  }
  this.Data["json"] = d
  this.ServeJSON()
}
```

你可以注意到我们的接口上有很多类似于 @Title, @Description 等的注解，这些注解可以帮助我们自动生成 api 文档。在使用 json.Unmarshal(this.Ctx.Input.RequestBody, &user) 时, 请确保 app.conf 文件中 copyrequestbody = true。

#### 配置路由

routers/router.go

```
// @APIVersion 1.0.0
// @Title beego Test API
// @Description beego has a very cool tools to autogenerate documents for your API
// @Contact astaxie@gmail.com
// @TermsOfServiceUrl http://beego.me/
// @License Apache 2.0
// @LicenseUrl http://www.apache.org/licenses/LICENSE-2.0.html
package routers

import (
    "my-api/controllers"
    "github.com/astaxie/beego"
)

func init() {
    ns := beego.NewNamespace("/v1",
        beego.NSNamespace("/user",
            beego.NSInclude(
                &controllers.UserController{},
            ),
        ),
    )
    beego.AddNamespace(ns)
}
```

routers/commentsRouter\_controllers.go

```
package routers

import (
    "github.com/astaxie/beego"
    "github.com/astaxie/beego/context/param"
)

func init() {
    beego.GlobalControllerRouter["my-api/controllers:UserController"] = append(beego.GlobalControllerRouter["my-api/controllers:UserController"],
        beego.ControllerComments{
            Method: "Post",
            Router: `/`,
            AllowHTTPMethods: []string{"post"},
            MethodParams: param.Make(),
            Filters: nil,
            Params: nil})

    beego.GlobalControllerRouter["my-api/controllers:UserController"] = append(beego.GlobalControllerRouter["my-api/controllers:UserController"],
        beego.ControllerComments{
            Method: "GetAll",
            Router: `/`,
            AllowHTTPMethods: []string{"get"},
            MethodParams: param.Make(),
            Filters: nil,
            Params: nil})

    beego.GlobalControllerRouter["my-api/controllers:UserController"] = append(beego.GlobalControllerRouter["my-api/controllers:UserController"],
        beego.ControllerComments{
            Method: "Get",
            Router: `/:uid`,
            AllowHTTPMethods: []string{"get"},
            MethodParams: param.Make(),
            Filters: nil,
            Params: nil})

    beego.GlobalControllerRouter["my-api/controllers:UserController"] = append(beego.GlobalControllerRouter["my-api/controllers:UserController"],
        beego.ControllerComments{
            Method: "Delete",
            Router: `/uid`,
            AllowHTTPMethods: []string{"delete"},
            MethodParams: param.Make(),
            Filters: nil,
            Params: nil})

}
```

#### 工具方法

utils/pagination.go

```
package utils

var (
    PageSize int = 10
)

type Pagination struct {
    Page        int             `json:"page"`
    PageSize    int             `json:"pageSize"`
    TotalPage   int             `json:"totalPage"`
    TotalCount  int             `json:"totalCount"`
    FirstPage   bool            `json:"firstPage"`
    LastPage    bool            `json:"lastPage"`
    List        interface{}     `json:"list"`
}

func Paginate(count int, page int, pageSize int, list interface{}) Pagination {
    tp := count / pageSize
    if count%pageSize > 0 {
        tp = count/pageSize + 1
    }
    return Pagination{Page: page, PageSize: pageSize, TotalPage: tp, TotalCount: count, FirstPage: page == 1, LastPage: page == tp, List: list}
}
```

#### 主函数

```
package main

import (
    _ "my-api/routers"
    "github.com/astaxie/beego"
    "my-api/controllers"
)

func main() {
    # 挂载 api doc
    if beego.BConfig.RunMode == "dev" {
        beego.BConfig.WebConfig.DirectoryIndex = true
        beego.BConfig.WebConfig.StaticDir["/swagger"] = "swagger"
    }

   # 注册 error 处理 controller
    beego.ErrorController(&controllers.ErrorController{})
    beego.Run()
}
```

### 总结

最终我们的项目结构为

```
.
|____routers
| |____commentsRouter_controllers.go
| |____router.go
|____database
| |____migrations
| | |____20200429_231223_users.go
|____go.mod
|____utils
| |____pagination.go
|____go.sum
|____models
| |____models.go
| |____user.go
|____controllers
| |____error.go
| |____user.go
|____conf
| |____app.conf
|____migration
|____main.go
```