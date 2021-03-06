# 数据中心
## 概述
数据中心处理并落地平台上下级停车数据、缴费数据、交易流水等数据，为小程序、H5、路侧停车缴费提供数据来源，为平台各种数据报表和可视化和统计数据提供接口；提供平台停车数据接出到第三方平台；等等等。
## 主要功能模块
### 停车数据接入
#### 路侧停车数据接入
1. 接入
> 车辆进出事件发生后，通过kafka消息接入停车事件，根据dataCenterId作为相同进出车事件判断条件
2. 处理
> 生成订单号；合并交易流水号
#### 室内停车数据接入
1. 接入
> 下级产生进出车事件后，通过调用上级http接口同步数据至上级平台
2. 处理
> 匹配相同进出车事件：根据seqCode匹配，如果没有seqCode，根据下级平台编号、停车场编号、车牌号、入场时间匹配，如果入场时间为空，根据下级数据ID匹配

> 更新处理：第一次进车生成订单号；重复进车以后进车为准，但图片单独处理，出车后匹配到本次入车数据只更新图片

> 图片处理：重复进车根据后一个进车数据有图片才更改原有数据图片，否则不更改处理，如果出车匹配到本次入车只处理图片，并且按同样规则处理图片。已有图片删除统一异步处理。

> 泊位更新：刷新车场泊位缓存
### 停车数据接出
1. 接出到支付宝无感支付
> 数据中心接入停车数据后，通过kafka消息同步数据至支付宝
2. 接出到其他第三方平台
> 数据中心接入停车数据后，通过kafka消息同步数据至对外API服务
### 支付数据接入
#### 路侧支付数据接入
1. 在场缴费数据接入
> 通过kafka消息接入在场缴费信息，得到缴费方式等，计算已交费用和最新费用差值得到最新缴费金额
2. 离场缴费数据接入
> 通过http接口接入离场缴费信息，得到缴费方式等，根据欠费总金额得到缴费金额
#### 室内支付数据接入
1. 接入
> 通过http接口接入支付详细数据
2. 处理
> 匹配对应停车数据：根据seqCode匹配，如果没有seqCode，根据下级平台编号、停车场编号、车牌号、入场时间匹配，如果入场时间为空，根据下级数据ID匹配

> 匹配重复调用事件：根据dataCenterId和交易流水匹配

> 更新停车数据：更新对应停车数据最后支付时间、已付金额、交易流水号等
### 交易流水
#### 路侧支付交易流水
通过http接口接入数据中心，匹配对应停车数据
#### 室内支付交易流水
1. 非现金支付非储值用户扣款
> 通过http接口接入数据中心，匹配对应停车数据
2. 现金支付或储值用户扣款
> 通过缴费详情同步接口接入数据中心，数据中心判断如果为现金支付或储值用户扣款方式，生成交易流水
### 停车原始记录
#### 下级停车原始记录
1. 记录生成
> 下级同步进出车记录后生成，通过dataCenterId与停车订单关联
2. 订单号更新
> 下级同步出场停车记录后，根据dataCenterId匹配入场停车记录，如果匹配成功，更新订单号为停车订单订单号
#### 路侧停车原始记录
1. 接入
> 数据中心通过http接口接入停车记录，通过dataCenterId与停车订单关联，重复接入执行更新
2.订单号生成
> 接收到停车出场数据时生成：根据dataCenterId查询完整停车记录，如果能查询到，更新停车记录订单号为停车订单订单号

> 接收到原始出场停车记录时生成：根据dataCenterId查询对应入场原始停车记录，如果查询到，根据dataCenterId查对应停车订单，更新停车记录订单号为停车订单订单号
### 小程序、H5、支付查询数据、第三方主动查询
提供根据车牌、在离场状态、车场、公司等多种场景查询的多种接口
### 数据明细、统计和可视化
1. 历史统计数据查询
> 历史数据通过定时任务预统计，接口查询直接从预统计表查询
2. 类型
> [营收明细](http://citypark-dev.cloud-dahua.com/#/revenue/revenueDetail)、[营收统计](http://citypark-dev.cloud-dahua.com/#/revenue/revenueStatistic)、[魔墙](http://citypark-dev.cloud-dahua.com/#/overview/dataOverview)、
[停车欠费管理](http://citypark-dev.cloud-dahua.com/#/workbench/parkingArrearsManager)等
