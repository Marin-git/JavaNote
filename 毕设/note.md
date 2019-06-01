[TOC]

### 毕设随笔
#### 参考资料
Gson使用方法：https://www.cnblogs.com/qinxu/p/9504412.html
Jedis方法Api：http://tool.oschina.net/uploads/apidocs/redis/clients/jedis/Jedis.html

#### 登录态
使用code换回来的session_key和openId，以session_key为key，openId为value，存储在Redis中，并设置过期时间。由于微信接口传回来的session_key的具体过期时间腾讯没说，所以我自己的过期时间设为1小时，即1小时内没操作则过期。
在小程序的请求头中加入session_key，以作为用户的登录凭证

#### 验证码
参考资料：https://blog.csdn.net/qq_39135287/article/details/80907606
使用秒滴短信平台，调用对方api，实现验证码的发送
此平台好处：
- 注册可以网上找一个带公章的营业执照就可以了
- 新用户免费送10元短信费，每条短信6分钱，足够测试使用了
- 有相关的示例demo，方便快速使用该接口

验证码需要自己生成，即可以随意设置验证码位数，
对于验证码的存储问题，主体是两种方案：
1. 后台存储：数据库
优点：
- 后台可随时做数据对比，无需前端传值
- 操作较简单（对于小程序来说，放数据库比较简单）
- 持久化验证码，对验证码的输入时间就可以宽松很多
- 经得起压力

缺点：
- 数据库需要多一个字段存储验证码

若有缓存（Redis）的话就好很多了，速度快，过期时间容易设置

2. 前台存储：session
说白了还是后台存储，因为前段只是存储session的id而已，session的主体还是存在数据库的
优点：
- 节省数据库字段

缺点：
- 对高并发的系统承受不了
- 需要设计过期时间
- 需要浏览器支持（因为需要cookie啊）

