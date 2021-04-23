# huobi_intf
提供火币网的获取数据的接口，可用于实盘交易和模拟交易

get_price接口得到火币的币的数据，返回dataframe的格式

info接口返回btc和eth的实时close价格，在浏览器里输入http://ip:8005/info可以看到网页格式的数据

接口说明:
1、取得火币网币的数据的接口/get_price

code='btc.usdt'
print(code,'最新价格',get_last_price(code))
df=get_price(code,count=5,frequency='4h');      #1d:1天  4h:4小时   60m: 60分钟    15m:15分钟
print(df)

btc.usdt 最新价格 53609.16

                      open	    close   	high       	low	          vol	
2021-04-22 00:00:00	56153.34	55479.34	56332.98	55141.91	1.727785e+08
2021-04-22 04:00:00	55479.33	53794.76	55479.34	53635.09	2.322080e+08
2021-04-22 08:00:00	53794.81	53506.68	54641.56	52577.19	5.136929e+08
2021-04-22 12:00:00	53506.68	54141.42	54850.00	53456.72	2.157972e+08
2021-04-22 16:00:00	54144.03	53609.17	54260.05	53609.16	4.257941e+07
