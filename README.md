# 君邦环境监测报警引擎




## 系统架构

```mermaid
graph LR
	recv("HJ-212 解析器") --> rt-queue(实时数据队列) & min-queue(分钟数据队列) & hour-queue(小时数据队列) & day-queue(日数据队列)
	subgraph RabbitMQ
		rt-queue
		min-queue
		hour-queue
		day-queue
	end
	rt-queue --> rt-consumer("实时数据处理：<br>1. 回写数据库<br>2. 推送给报警引擎")
	min-queue --> min-consumer("分钟数据处理：<br>1. 回写数据库<br>2. 推送给报警引擎")
	hour-queue --> hour-consumer("小时数据处理：<br>1. 回写数据库")
	day-queue --> day-consumer("日数据处理：<br>1. 回写数据库")
	rt-consumer & min-consumer --> alert-engine("报警规则引擎") --> alert-exec("报警执行引擎")
  rt-consumer & min-consumer & hour-consumer & day-consumer --> database[(MySQL)]
	
  database & alert-engine --- curr-alert("现有的报警服务")
  style curr-alert stroke-dasharray: 5 5, stroke-width: 2, fill: #bbf
```



## 报警规则引擎

```mermaid
graph LR
	monitoring-data("监测数据") --> data-coding("报警数据编码")
	data-coding --> rule-engine("运行报警规则")
```

### 监测数据格式

**实时数据格式：**

| 序号 | 名称     | 描述                           |
| ---- | -------- | ------------------------------ |
| 1    | mn       | 站点编码                       |
| 2    | datatime | 数据时间                       |
| 3    | code     | 监测因子编码（数采仪原始编码） |
| 4    | value    | 监测值                         |

**分钟数据格式：**

| 序号 | 名称     | 描述                           |
| ---- | -------- | ------------------------------ |
| 1    | mn       | 站点编码                       |
| 2    | datatime | 数据时间                       |
| 3    | code     | 监测因子编码（数采仪原始编码） |
| 4    | min      | 最小值                         |
| 5    | max      | 最大值                         |
| 6    | avg      | 平均值                         |



### 报警数据编码数据结构

```json
{
  "mn-1": {
    "rtValueSameAsPrev": false,
    "minDataOverAlarm": false,
    "minDataOverStandard": false,
    "minDataOutOfRange": false
  }
}
```

