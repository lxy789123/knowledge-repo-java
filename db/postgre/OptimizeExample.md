# 优化示例
## ー、问题描述
### 1.目标sql
```sql
<select id="queryTxcodeMapInfoByssCodeAndTxcodes" resultMap="txcodeMapInfo">
select a. front_txcode, a. front_txname, a. flag, a.pub_authen_method, a.special_authen_method, a.financial_authen_method, 
t.outsys_txcode_id,
k.outsys_txcode, k.outsys_txname, k.create_sys_code, k.rel_sys_addr, k.msg_agr-type, k.comm_agr_type, k.req_type, k.msgrpt_type_no, k.path, k.resp_code_path, k.resp_info_path, k.inst_no, k.creator 
from th_ front_ txcode_ info a
FULL JOIN th_txcode_map_info t on a. front_txcode = t. front_txcode
FULL JOIN th_outsys_txcode_info k on k.outsys_txcode_id = t.outsys_txcode_id
where k. create_sys_code=# IsysCodel and a. Front_txcode in
    <foreach item-"item" collection-"txcodes" open="(" separator=","close=")">
        {#item, jdbcType=VARCHAR}
    </foreach>
</select>
```
### 2.表结构
#### 2.1.tb front_txcode_info as a
|     字段名称     |     类型     |                      备注                      |
| :--------------: | :----------: | :--------------------------------------------: |
| **front_txcode** | varchar(11)  |                   前置交易码                   |
|   front_txname   | varchar(120) |                  前置交易名称                  |
|  security_level  |  varchar(3)  | 安全等级 10: 证书  20: SM3 100: MAC 99: 不加密 |
|       flag       |   char(1)    |       启停标志，需要授权 0：停用 1：启用       |

索引详情：

"tb_front_txcode_info_key" PRIMARY KEY("front_txcode");

#### 2.2、 tb_txcode_map_info as t

|       字段名称       |    类型     |        备注        |
| :------------------: | :---------: | :----------------: |
|   **front_txcode**   | varchar(11) |     前置交易码     |
| **outsys_txcode_id** | varchar(32) | 外系统交易码自增id |

索引详情：

"tb_txcode_map_info_pkey" PRIMARY KEY("frront_txcode","outsysy_txcode_id");

"uk_front_txcode" UNIQUE("front_txcode");

"uk_outsys_txcode_id" UNIQUE("outsys_txcode_id");

#### 2.3、 tb_outsys_txcode_info as k

|      字段名      |     类型     |             备注             |
| :--------------: | :----------: | :--------------------------: |
| outsys_txcode_id | varchar(32)  |           自增主键           |
|  outsys_txcode   | varchar(50)  |         外系统交易码         |
| create_sys_code  | varchar(12)  |          接出系统号          |
|   rel_sys_addr   | varchar(255) | 接出系统地址（ip:port//url） |
|   msg_agr_type   |  varchar(2)  |   报文类型 0：json 1：xml    |
|  comm_agr_type   |  varchar(2)  |   传输协议 0：http 1：tcp    |
|     req_typq     |   char(1)    |   请求类型 0：post 1：get    |
|  msgrpt_type_no  |   char(1)    |     编码 0：urf-8 1：gbk     |
|  outsys_txname   | varchar(120) |        外系统交易名称        |
|       path       | varchar(150) |    交易码映射路径 $.a.b.c    |
|  resp_code_path  | varchar(150) |        响应码映射路径        |
|  resp_info_path  | varchar(150) |       响应信息映射路径       |
|     inst_no      | varchar(12)  |            机构号            |

索引详情：

"tb_outsys_txcode_info_pkey" PRIMARY KEY("outsys_txname", "create_sys_code");

## 二、分析

### 1.sql执行过程

sql示例：

```sql
explain
select a.front_txcode, a.front_txname, a.flag, a.pub_authen_method,
	   a.special_authen_method, a.financial_authen_method,
	   t.outsys_txcode_id,
	   k.outsys_txcode, k.outsys_txname, k.create_sys_code,
	   k.rel_sys_addr, k.msg_agr_type, k.comm_agr_type,
	   k.req_type, k.msgrpt_type_no, k.path, k.resp_code_path,
	   k.resp_info_path, k.inst_no, k.creator
	   from tb_front_txcode_info a
	   FULL JOIN tb_txcode_map_info t on a.front_txcode = t.front_txcode
```

explain结果：

```
Nested Loop (cost=9.12..31.98 rows=1 width=192)
	-> Nested Loop (cost=8.84..31.07 rows=2 width=53)
		-> Bitmap Heap Scan on tv_front_txcode_info a (cost=8.57..14.47 rows=2 width=46)
			Recheck Cond: ((front_txcode)::text = ANY('{123.234}'::text[]))
			-> Bitmap Index Scan on tv_front_txcode_info_pkey (cost=0.00..8.57 row2=2 width=0)
				Index Cond: ((front_txcode)::text =ANY ('{123,234}'::text[]))
		-> Index Only Scan using tb_txcode_map_info_pkey on tb_txcode_map_info t (cost=0.28..8.29 rows=1 width=14)
			Index Cond: (front_yxcode =(a.front_txcode)::text)
	-> Index Scan using idx_outsys_txcode_id on tv_outsys_txcode_info k (cost=0.28..0.45 rows=1 width=146)
		Index Cond: ((outsys_txcode_id)::text = (t.outsys_txcode_id)::text)
		Filter:((create_sys_code)::text = '123'::text)
```

