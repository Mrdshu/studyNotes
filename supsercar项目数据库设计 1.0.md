#supsercar项目数据库设计 1.0

现粗略将系统分为三大模块功能：（后面画数据流图进一步细化）

1. 维修工单模块：主要车辆的维修开单功能。可参考购物网站交易模块功能
2. 库存操作模块：配件的库存管理，出入库操作。可参考购物网站商品模块功能
3. 用户模块：后台用户以及客户信息的维护。可参考购物网站用户模块功能



预计开发顺序：由简至难，用户模块——》库存模块——》维修工单模块。

粗略考虑后的数据库表如下，标蓝部分为数据字典表，其余为主表。
（初版一个人考虑必然有所不周，如有遗漏，请用<font color = 'red'>标红字体</font>帮忙检查补充）

## **维修工单模块（交易）**

###1. 维修工单表(M_WORK_ORDER)

|字段名 |中文名 |类型   |描述|
|:-----:|:-----:|:-----:|:-----:|
|id     | id    |String|主键|
|mwo_workorder_num     | 工单号          |String |
|mwo_workorder_repairs     | 修理性质          |String |
|mwo_workorder_state     | 工单状态          |String |
|mwo_workorder_money     | 结算金额          |String |
|mwo_workorder_clerk     | 服务顾问          |String |
|mwo_workorder_starttime     | 送修时间          |Date |
|mwo_workorder_endtime     | 交车时间          |Date |
|mwo_car_no     | 车牌号          |String |
|mwo_car_framework     | 车架号          |String |
|mwo_car_model     | 车型            |String | “车型”数据字典表的外键
|mwo_car_mileage     | 里程            |String |
|mwo_car_oil_meter     | 进店油表        |String |
|mwo_car_insurance_endtime     | 保险到期时间    |String |
|mwo_client_name     | 客户姓名        |String |
|mwo_client_mobile     | 手机号          |String |
|mwo_client_advice    | 车主嘱咐        |String |
|extend1    | 预留拓展字段        |String |
|extend2    | 预留拓展字段        |String |
|extend3    | 预留拓展字段        |String |

###2.1 维修项目表（M_ITEM）
|字段名 |中文名 |类型   |描述|
|:-----:|:-----:|:-----:|:-----:|
|id     | id    |String|主键|
|mit_id     | 项目类型    |String|“项目类型”数据字典表的外键|
|mi_code     | 项目代码    |String|
|mi_name     | 项目名称    |String|
|mi_work_time     | 工时数    |String|
|mi_work_type     | 工种    |String|“项目工种”数据字典表的外键|
|mi_desc     | 备注    |String|
|mi_sum     | 金额    |String|
|extend1    | 预留拓展字段        |String |
|extend2    | 预留拓展字段        |String |
|extend3    | 预留拓展字段        |String |

###2.2 <font color = 'blue'>维修项目类型表（LOOKUP_MITEM_TYPE）</font>
|字段名 |中文名 |类型   |描述|
|:-----:|:-----:|:-----:|:-----:|
|id     | id    |String|主键|
|code     | 项目类型code    |String||
|name     | 项目类型名称    |String||

###2.3 <font color = 'blue'>维修项目工种表（LOOKUP_MITEM_WORK_TYPE）</font>
|字段名 |中文名 |类型   |描述|
|:-----:|:-----:|:-----:|:-----:|
|id     | id    |String|主键|
|code     | 工种code    |String||
|name     | 工种名称    |String||

###3.1 维修工单-项目表 中间表（MWO_MI）
|字段名 |中文名 |类型   |描述|
|:-----:|:-----:|:-----:|:-----:|
|id     | id    |String|主键|
|mwo_id     | 维修工单id    |String|“维修工单”外键|
|mi_id     | 维修项目id |String|“维修项目”外键|
|wi_mechanic| 维修工 |String|“用户”外键|
|wi_starttime|开始时间|Date|
|wi_endtime|结束时间|Date|
|extend1    | 预留拓展字段        |String |
|extend2    | 预留拓展字段        |String |
|extend3    | 预留拓展字段        |String |

###3.2 维修工单-配件表 中间表（MWO_MPARTS）
|字段名 |中文名 |类型   |描述|
|:-----:|:-----:|:-----:|:-----:|
|id     | id    |String|主键|
|mwo_id     | 维修工单id    |String|“维修工单”外键|
|mp_id     | 配件id |String|“配件”外键|
|mp_count     | 数目 |String|领料数目|
|extend1    | 预留拓展字段        |String |
|extend2    | 预留拓展字段        |String |
|extend3    | 预留拓展字段        |String |


## **用户模块（用户信息）**

###1.1 用户表（u_user）
|字段名 |中文名 |类型   |描述|
|:-----:|:-----:|:-----:|:-----:|
|id     | id    |String|主键|
|u_username     | 用户名    |String|
|u_fullname     | 全名   |String|
|u_password     | 密码    |String|
|u_email     | 邮箱    |String|
|u_mobile     | 电话    |String|
|u_company     | 分公司    |String|
|u_role     | 角色    |String|“角色”表外键
|isDisable    | 禁用    |Boolean|
|extend1    | 预留拓展字段        |String |
|extend2    | 预留拓展字段        |String |
|extend3    | 预留拓展字段        |String |

###1.2 <font color = 'blue'>角色表（LOOKUP_USER_ROLE）</font>
|字段名 |中文名 |类型   |描述|
|:-----:|:-----:|:-----:|:-----:|
|id     | id    |String|主键|
|code     | 角色code    |String||
|name     | 角色名称    |String||

###2.1 客户表（u_client）
|字段名 |中文名 |类型   |描述|
|:-----:|:-----:|:-----:|:-----:|
|id     | id    |String|主键|
|c_name     | 客户姓名   |String|
|c_sex     | 客户性别   |Boolean|
|c_idcard     | 客户身份证   |String|
|c_type     | 客户类别   |String|“客户类别”表外键
|c_email     | 邮箱    |String|
|c_mobile     | 手机号    |String|
|c_address     | 地址    |String|
|create_time     | 创建日期            |Date |
|update_time     | 更新日期            |Date |
|isDeleted     | 软删除标志   |Boolean|
|extend1    | 预留拓展字段        |String |
|extend2    | 预留拓展字段        |String |
|extend3    | 预留拓展字段        |String |
|car_no     | 车牌号          |String | 唯一值
|car_model     | 车型            |String | “车型”数据字典表的外键
|car_chassis     | 底盘号            |String |
|car_color     | 车身颜色            |String |
|car_engine     | 发动机号            |String |
|car_insurer     | 保险公司            |String |
|car_insurance_endtime     | 保险到期时间            |Date |
|car_registration_time     | 上牌日期            |Date |
|create_time     | 创建日期            |Date |
|update_time     | 更新日期            |Date |
|isDeleted     | 软删除标志   |Boolean|

###2.2 客户类别表（u_client_type）
|字段名 |中文名 |类型   |描述|
|:-----:|:-----:|:-----:|:-----:|
|id     | id    |String|主键|
|code     | 客户类别code    |String||
|name     | 客户类别名称    |String||
|description     | 描述    |String||
|parts_discount     | 配件折扣    |double||
|items_discount     | 项目工时折扣    |double||


###5. 门店表（待定）

###6. 公司信息表（待定）


## **配件模块（商品）**

###1.1 维修配件表（P_PARTS）
|字段名 |中文名 |类型   |描述|
|:-----:|:-----:|:-----:|:-----:|
|id     | id    |String|主键|
|p_code         | 配件编号      |String|
|p_name         | 配件名称        |String|
|p_unit         | 单位        |String|"配件单位"数据字典外键|
|p_sale         | 销售价      |String|
|p_wholesale         | 批发价      |String|
|p_produce_area    | 产地        |String|
|p_specification     | 规格        |String|
|p_car_model     | 适用车型    |String|"车型"数据字典外键（多值）
|p_category     | 分类        |String|“配件分类”数据字典外键
|isdisable     | 有效        |Boolean|
|extend1    | 预留拓展字段        |String |
|extend2    | 预留拓展字段        |String |
|extend3    | 预留拓展字段        |String |

###1.2 <font color = 'blue'>维修配件分类表（LOOKUP_PARTS_TYPE）</font>
|字段名 |中文名 |类型   |描述|
|:-----:|:-----:|:-----:|:-----:|
|id     | id    |String|主键|
|code     | 配件code    |String||
|name     | 配件名称    |String||

###1.3 <font color = 'blue'>维修配件单位表（LOOKUP_PARTS_UNIT）</font>
|字段名 |中文名 |类型   |描述|
|:-----:|:-----:|:-----:|:-----:|
|id     | id    |String|主键|
|code     | 单位code    |String||
|name     | 单位名称    |String||

###1.4 <font color = 'blue'>车型表（LOOKUP_CAR_MODEL）</font>
|字段名 |中文名 |类型   |描述|
|:-----:|:-----:|:-----:|:-----:|
|id     | id    |String|主键|
|code     | 车型code    |String||
|name     | 车型名称    |String||
|description     | 描述    |String||
###2.1 库存表（P_INVENTORY）
|字段名 |中文名 |类型   |描述|
|:-----:|:-----:|:-----:|:-----:|
|id     | id    |String|主键|
|p_id     |配件id    |String|“配件”表外键
|p_count     | 配件库存数目    |String|
|p_cost         | 进货价      |String|
|p_supplier       | 供应商      |String|“供应商”数据字典外键|
|r_code     | 库位号   |String|“库位”数据字典外键
|extend1    | 预留拓展字段        |String |
|extend2    | 预留拓展字段        |String |
|extend3    | 预留拓展字段        |String |

###2.2 <font color = 'blue'>库位表（LOOKUP_REPOSITORY）</font>
|字段名 |中文名 |类型   |描述|
|:-----:|:-----:|:-----:|:-----:|
|id     | id    |String|主键|
|code     | 库位code    |String||
|name     | 库位名称    |String||

###2.3 <font color = 'blue'>供应商表（LOOKUP_SUPPLIER）</font>
|字段名 |中文名 |类型   |描述|
|:-----:|:-----:|:-----:|:-----:|
|id     | id    |String|主键|
|code     | 供应商code    |String||
|name     | 供应商名称    |String||

###3.1 入库工单表（P_IN_PARTS）
|字段名 |中文名 |类型   |描述|
|:-----:|:-----:|:-----:|:-----:|
|id     | id    |String|主键|
|pin_workorder_num     | 入库单号    |String|
|pin_time     | 入库时间    |Date|
|p_supplier       | 供应商      |String|“供应商”数据字典外键|
|pin_pay_method     | 结算方式    |String|
|pin_money    | 合计金额    |String|
|extend1    | 预留拓展字段        |String |
|extend2    | 预留拓展字段        |String |
|extend3    | 预留拓展字段        |String |

###3.2 入库配件表（P_IN_PARTS_INFO）
|字段名 |中文名 |类型   |描述|
|:-----:|:-----:|:-----:|:-----:|
|id     | id    |String|主键|
|pin_workorder_num     | 入库单号    |String|
|p_id     |配件id    |String|“配件”表外键
|p_count     | 配件入库数目    |String|
|p_cost         | 进货价      |String|
|p_supplier       | 供应商      |String|“供应商”数据字典外键|
|r_code     | 库位号   |String|“库位”数据字典外键
|extend1    | 预留拓展字段        |String |
|extend2    | 预留拓展字段        |String |
|extend3    | 预留拓展字段        |String |


###4.1 出库表（P_OUT_PARTS）
|字段名 |中文名 |类型   |描述|
|:-----:|:-----:|:-----:|:-----:|
|id     | id    |String|主键|
|pout_workorder_num     | 出库单号    |String|
|pout_time     | 出库时间    |Date|
|pout_type     | 出库类型    |Date|
|pout_client_name    | 车主名称    |String|
|pout_department    | 部门名称    |String|“部门”表外键
|pout_getMan    | 领料人    |String|“用户”表外键
|pout_money    | 合计金额    |double|
|extend1    | 预留拓展字段        |String |
|extend2    | 预留拓展字段        |String |
|extend3    | 预留拓展字段        |String |

###4.2 出库配件表（P_IN_PARTS_INFO）
|字段名 |中文名 |类型   |描述|
|:-----:|:-----:|:-----:|:-----:|
|id     | id    |String|主键|
|pout_workorder_num     | 出库单号    |String|
|pout_id     |库存配件id    |String|“库存”表外键
|pout_count     | 配件出库数目    |String|
|extend1    | 预留拓展字段        |String |
|extend2    | 预留拓展字段        |String |
|extend3    | 预留拓展字段        |String |

###5. 优惠券表（待定）
