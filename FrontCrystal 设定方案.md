<!--
title: FrontCrystal 设定方案
date: 2017-11-19 17:12
-->
# FrontCrystal 设定方案
## 结构
### Shadowsocks 衔接
#### Manager Api
> 使用 Shadowsocks-libev & Manager Api   
> Manager Mode 能进行 ping、add、delete，但数据结构单一，且不能持久保存。

server (manager mode) <=> crystal_collect <=> crystal_center

#### crystal_collect
> crystal_collect 充当中介；   
> 具有和 server 沟通，进行数据使用统计、操作的功能；   
> 具有和 center 沟通，提供 Api 供其检测服务可用性、统计、操作的功能
##### 数据储存
> MySQL 存储；   
> “data_usage” 表中 period_usage 计算本次统计距离上次统计的数据用量；   
> “server” 表 中 data_usage 和 “data_usage” 表中  totoal_usage 按自然月统计；   
> 自然月重新开始不影响 “data_usage” 表中 period_usage 统计

"**servers**" 表
Column	        | Type   
--------------- | -----------------------
id	      	    | tinyint(3) unsigned AI
port	   	 	| tinyint(3) unsigned
password		| varchar(60)	 
data_usage | bigint(20) unsigned
last_activity	| timestamp	 
created_date	| timestamp	 
modified_date	| timestamp   
forbidden		| tinyint(1) unsigned

"**data_usage**" 表
Column	        | Type   
--------------- | -----------------------
id	      	    | tinyint(3) unsigned AI
port	   	 	| tinyint(3) unsigned
collect_at      | timestamp
period_usage    | bigint(20) unsigned
totoal_usage    | bigint(20) unsigned

**数据统计周期**
间隔	        | 执行  
--------------- | -----------------------
1 分钟	      	| data_usage 统计 <=> server 表

#### Api
> HTTP Restful Api    
> 一切操作由 center 负责

**url**    
http://{ss-server}.yunet.work/api/{do}?key={api-key}  
**返回总概**   	  
GET /data/all   
```json
[
	{
		"port": 1234,
		"password": b21a8913917h,
		"data_usage": 1123123213,
		"last_activity": 2017-12-25 11:19:20,
		"created_date": 2017-12-25 01:12:11,
		"modified_date": 2017-12-25 01:12:11,
		"forbidden": 0
	}, {
		...
		....
		.....
	}
]
```

**返回单个端口总概**   
GET /data/{port}    
例：  GET /data/1234
```json
{
	"port": 1234,
	"password": b21a8913917h,
	"data_usage": 1123123213,
	"last_activity": 2017-12-25 11:19:20,
	"created_date": 2017-12-25 01:12:11,
	"modified_date": 2017-12-25 01:12:11,
	"forbidden": 0
}
```

**返回全体数据使用**   
GET /data/all/analyze
```json
[
	{
		"port": 1234,
		"period_usage": [
			{
				"collect_at": 2017-12-25 01:12:11,
				"period_usage": 1232
			}, {
				....
				.....
			}
		],
		"totoal_usage": 1123123213
	}, {
		.....
	}
]
```
## 计费

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTk3OTMwNjAzMV19
-->