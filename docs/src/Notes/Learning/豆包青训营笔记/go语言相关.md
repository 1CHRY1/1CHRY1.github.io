## GO语言相关框架

#### <font color=darkblue>字节内部框架</font>

##### 三件套

gorm —— orm框架
kitex —— rpc框架
hertz —— http框架

##### gorm基本使用

文档：https://gorm.io/zh_CN/docs/index.html

*gorm约定*：

· 默认使用名为id的字段作为主键。
· 需定义表名，若未定义则将结构体名的蛇形复数作为表名
· 使用字段名的蛇形作为列名
· 使用CreatedAt，UpdateQAt字段作为创建和更新时间

*gorm支持的数据库*：
mySQL，SQLServer，PostgreSQL，SQLite

##### kitex基本使用

文档：https://www.cloudwego.io/zh/docs/kitex

示例代码：https://github.com/kitex-contrib/registry-etcd/tree/main/example/client

##### Hertz基本使用

demo项目：https://github.com/cloudwego/kitex-examples/tree/main/bizdemo/easy_note