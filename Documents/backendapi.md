# QUANTAXIS 的后台api


quantaxis 采用前后端分离的模式开发,所以对于后端而言 是一个可以快速替换/语言随意的部分.只需要按照规则设置好REST的url即可


## 后端的标准和格式规范

### 基础标准

quantaxis的后台可以用 nodejs(express/koa), python(flask/django/tornado), go 等等语言实现

quantaxis的后台部分主要是作为微服务,给前端(web/client/app)等提供标准化的查询/交互接口


## 命名格式

quantaxis的后台命名格式

http://ip:port/功能(backtest/marketdata/user/..)/细分功能(info/query_code/..)

example:

```
http://localhost:3000/backtest/info_cookie?cookie=xxxxx  ==>  按backtest的cookie查询backtest的信息

```

## 后端的实现方式和注意事项


### 跨域支持

因为是前后端分离的模式, 需要对于url采取跨域允许

跨域在python中的实现

#### Flask

```python
@app.route("/status")
def status():
    rst = make_response(jsonify('200'))
    rst.headers['Access-Control-Allow-Origin'] = '*'
    rst.headers['Access-Control-Allow-Methods'] = 'PUT,GET,POST,DELETE'
    allow_headers = "Referer,Accept,Origin,User-Agent"
    rst.headers['Access-Control-Allow-Headers'] = allow_headers
    return rst

```


#### Tornado

```python
class BaseHandler(tornado.web.RequestHandler):

    def set_default_headers(self):
        print（"setting headers!!!"）
        self.set_header("Access-Control-Allow-Origin", "*") # 这个地方可以写域名
        self.set_header("Access-Control-Allow-Headers", "x-requested-with")
        self.set_header('Access-Control-Allow-Methods', 'POST, GET, OPTIONS')

    def post(self):
        self.write('some post')

    def get(self):
        self.write('some get')

    def options(self):
        # no body
        self.set_status(204)
        self.finish()
```

跨域在nodejs中的实现

#### express

```javascript

router.get('*', function (req, res, next) {
  res.header("Access-Control-Allow-Origin", "*");
  res.header("Access-Control-Allow-Headers", "X-Requested-With");
  res.header("Access-Control-Allow-Methods", "PUT,POST,GET,DELETE,OPTIONS");
  res.header("X-Powered-By", ' 3.2.1')
  res.header("Content-Type", "application/json;charset=utf-8");
  next();
});

```

### 权限

后台服务需要保护好隐私不被泄露,避免路径攻击和端口暴露等问题

## 必须实现的部分


### 用户管理 /user

登陆

http://[ip]:[port]/users/login?name=[]&password=[]

注册

http://[ip]:[port]/users/signup?name=[]&password=[]


### 回测部分 /backtest



### 行情查询部分 /marketdata & /data

功能性的API,分别代表着 日线/分钟线/实时(5档)/分笔数据

#### URI总规则 GENERAL URI RULE
总URI为 http://[ip]:[port]/[market_type]/[frequence]?code=[]&start=[]&end=[]

#### 股票日线 STOCK DAY
http://[ip]:[port]/marketdata/stock/day?code=[]&start=[]&end=[]

当不给定结束日期的时候,返回的就是直到当前的数据

#### 股票分钟线 STOCK MINDATA
http://[ip]:[port]/marketdata/stock/min?code=[]&start=[]&end=[]

当不给定结束日期的时候,返回的就是直到当前的数据

#### 股票实时上下五档 STOCK REALTIME 5-ASK/BID
http://[ip]:[port]/marketdata/stock/realtime?code=[]

实时返回股票的L1上下五档的行情数据

#### 股票分笔数据 STOCK TRANSACTION
http://[ip]:[port]/marketdata/stock/transaction?code=[]&start=[]&end=[]

code 指的是具体的股票代码
start 指的是分笔开始的时间
end 指的是分笔结束的时间

#### 股票财务数据

http://[ip]:[port]/marketdata/stock/info?code=[]&time=[]

code 指的是具体的股票
time 指的是时间段

如 code=000001 time=2018Q1 指的是000001的2018年1季度

time的格式为: YEAR['YYYY']+Q+times[1,2,3,4](1- 1季度财报 2- 半年度财报 3- 3季度财报 4- 年报)

#### 期货日线
http://[ip]:[port]/marketdata/future/day?code=[]&start=[]&end=[]


#### 期货分钟线
http://[ip]:[port]/marketdata/future/min?code=[]&start=[]&end=[]

### 实时行情推送 /quotation

/quotation 推送指的是 建立一个websocket链接:

1. user login [Handler]

2. auth

3. send_req

4. make connection

5. data transport
