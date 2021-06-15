# hb_spark（数字币—火花： 开源，免费，稳定，实盘）

hb_spark提供火币网(币安在开发中)的高速免费实时行情服务器(支持火币网所有交易对的实时行情)，自带API缓存，可用于实盘交易和模拟回测。
行情数据，是一切量化交易的基础，可以获取1min、60min、4hour、1day等数据。数据能进行缓存，可以在多个币种，多个时间段查询的时候，查询速度依然很快。
服务框架采用基于强大的异步网络库tornado

### 火币网交易API最简封装（ https://github.com/mpquant/huobi_trade ）
* 包含现货交易，期货合约交易，配合本行情端，一套可应用于实盘的量化交易系统很快就能搭建起来   
   
### 功能特点（ hb_spark 开源版 ）
* 为什么要用独立的行情服务器，直接调用火币的行情API不行吗？( https://github.com/mpquant/huobi_hq 我们另外一个项目，直接调用火币行情api的封装) 
  火币行情服务器比较慢，尤其在国内被墙，无法访问，尤其多个策略运行，一次需要获取多个币种数据做实时指标计算的时候，慢的让你怀疑人生，无法满足实盘要求，所以必须架设独立的行情服务器

* 自带Redis接口缓存，重复的历史日线，4小时线等数据获取一次即被缓存，下次策略再取得时候直接返回，速度加速上百倍  

* 完美包装火币的行情API,行情服务对外只简洁到一个get_price()函数，可以实时获取火币所有交易对实时行情，包括按天，4小时，1小时，15分钟，5分钟，1分钟的实时行情

* 历史行情支持从当前时间往前推3000条，和历史任意时间200条的数据 (需要历史任意时间3000以上条可以联系下面微信）

* 在本行情服务器基础上开发有增强版的本地历史行情服务器内存数据版，用来做策略回测，调参等，可以达到半年数据1分钟跑完回测看结果 (感兴趣的可以联系下面微信）

* 最简函数调用 `get_price(security, start_date=None, end_date=None, count=None, frequency='1d', fields=['open','close', 'low', 'high'])` 具体看下面例子，懂的自然能看懂

* 行情服务器编程语言是python,采用高性能异步网络框架tornado做WebApi, 标准json返回，所有语言都能方便调用

* 经过半年的运行，已经是非常稳定的版本，可以直接用在实盘生成环境

* 针对高频交易，每秒都要更新价格那种，可以联系最下面微信，我们提供分布式多端密集更新方案，来满足秒级别行情的要求

* 稳定，可靠，及时的行情服务器是高波动率数字货币量化交易的基础实施,我们全部开源出来，希望能帮助到大家


### 高频实时行情服务器(huobi_High_frequency 收费版)
* 火币websocket行情模式，当关注币种价格数据变化，会主动推送过来(20ms频率)，我们采用多个接收端密集接收， 统一发送给 hb_spark行情服务器缓存，策略正常调用get_price()获取实时高频行情价格
   
   
### [数字货币回测行情服务器(huobi_backtest 收费版)](https://github.com/mpquant/huobi_backtest)
* 根据需求，自动提前下好N年N多币种历史数据(1分钟线),回测服务器自动转换1分钟线成为日线，4小时线，1小时等，并全部调入内存数据库，以便达到每个get_price()调用 小于 1.5ms的超高速历史回测行情，用最短的时间验证你的想法和策略 (从理论到实践，真金白银需要三思而行)   移步： https://github.com/mpquant/huobi_backtest
   

### 系统架构图：
<div  align="center"> <img src="/img/构架图.jpg" width = "600" height = "400" alt="火币行情服务器架构" /> </div>

### 手动部署服务
> 安装好第三方库后，运行文件huobi_app.py启动服务
```  
python3 huobi_app.py              #(可选运行参数  --port=8005  )
```       

#### docker模式一键部署运行
> 用下面一行命令在服务器运行镜像,默认端口是8005 (服务器尽量采用linux )，默认的火币api地址为api.huobi.de.com
```  
docker run -d --restart=always --net host --name huobi_intf  mpquant/huobi_intf
```
> 如过要修改火币api地址用下面一行命令在服务器运行镜像,加入-e HBADDR=api.hadax.com，可以修改api地址
```  
docker run -d --restart=always --net host -e HBADDR=api.hadax.com --name huobi_intf  mpquant/huobi_intf
```
启动成功后，在浏览器里输入`http://127.0.0.1:8005/info`，如果能出现下边的画面，说明启动成功了  
<div  align="center"> <img src="/img/info.png" width = "400" height = "150" /> </div>


## 需安装第三方库 （python >= 3.6 ）
* requests
* pandas
* tornado
* websocket-client

安装时执行命令:pip install -r requirements.txt

### 接口说明
* get_price接口得到火币的币的数据，返回dataframe的格式

* 为了支持多交易所(币安，okex等),我们规范定义了几个核心数据格式
   * 交易对统一用 btc.usdt  ，  eth.usdt  ，    eth.btc  这样的中间加.分割的格式
   * 时间周期统一用 1d: 一天 ，  4h: 四小时 ，  60m: 60分 ，  15m:15分 ，  5m:5分 ，   1m:1分   这样的格式

> 例子演示文件 `intf_test.py`
```python
#1分钟的数据获取
df = get_price('btc.usdt', end_date='2021-04-25 18:56:23', count=1, frequency='1m')

#日线的数据获取
df = get_price('btc.usdt', end_date='2021-04-25 18:56:23', count=10, frequency='1d')

#4小时的数据获取
df = get_price('btc.usdt', end_date='2021-04-25 18:56:23', count=10, frequency='4h')

#1小时的数据获取
df = get_price('btc.usdt', end_date='2021-04-25 18:56:23', count=10, frequency='60m')

```

## btc的1分钟线
![btc1min](/img/btc_1min.png)

## btc日线
![btc日线](/img/btc_1day.png)
 

### 实盘策略收益图
下图是利用利用这个行情框架，跑的策略收益图 (山寨币趋势轮动策略)

  <div  align="center"><b>策略思想：15个主流币种轮动，选涨幅最好的前3个币种买入</b></div>
  <div  align="center"> <img src="https://github.com/mpquant/huobi_backtest/blob/main/img/H103.jpg" width = "1200" height = "480" /> </div>



----------------------------------------------------

### 团队其他开源项目
* [MyTT 通达信,同花顺公式指标，文华麦语言的python实现](https://github.com/mpquant/MyTT)

* [Hb_Spark数字货币高速免费实时行情服务器,量化必备](https://github.com/mpquant/huobi_intf)

* [hb_trade火币网交易接口API最简封装,提供现货期货合约](https://github.com/mpquant/huobi_trade)

* [backtest数字货币历史回测服务器,高速内存数据库实现](https://github.com/mpquant/huobi_backtest)

* [ths_trade同花顺自动化交易股票下单接口API,量化框架](https://github.com/mpquant/ths_trade)

* [Ashare最简股票行情数据接口API,A股行情完全开源免费](https://github.com/mpquant/Ashare)



## 巴特量化
* 数字货币 股市量化工具 行情系统软件开发
* BTC虚拟货币量化交易策略开发 自动化交易策略运行
----------------------------------------------------

![加入群聊](/img/qrcode.png) 

> #### 数字货币量化交易研究大群, 股市程序化交易, 圈内大咖量化策略分享
> #### 全是干货，无闲聊 ，物以类聚, 人以群分，一起感受思维碰撞的力量 !
