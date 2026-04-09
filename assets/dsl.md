# Toco 设计元素 DSL 手册

本文档从 Toco 平台知识库中抽取各类设计元素的 DSL 表达方式，方便在设计、评审、代码生成和排查时快速查阅。

每个设计元素统一包含三部分：
- 作用说明：这个元素主要解决什么问题、通常放在哪一层使用
- DSL 示例：如何表达
- 生成代码：会落到哪些代码文件，以及这些文件的职责

## 目录
- [1. 枚举（Enum）](#1-枚举enum)
- [2. 值对象（EO）](#2-值对象eo)
- [3. 实体（Entity）](#3-实体entity)
- [4. 聚合对象（BO）](#4-聚合对象bo)
- [5. 数据传输对象（DTO）](#5-数据传输对象dto)
- [6. 视图对象（VO）](#6-视图对象vo)
- [7. 读方案（ReadPlan）](#7-读方案readplan)
- [8. 写方案（WritePlan）](#8-写方案writeplan)
- [9. 服务层方法（RPC / SERVICE）](#9-服务层方法rpc--service)
- [10. API](#10-api)
- [11. 流程服务（FunctionFlow）](#11-流程服务functionflow)
- [12. 领域消息（DomainMessage）](#12-领域消息domainmessage)
- [13. 普通消息（CommonMessage）](#13-普通消息commonmessage)
- [14. 订阅消息（SubscribeMessage）](#14-订阅消息subscribemessage)

## 1. 枚举（Enum）

### 作用说明
Enum 用于表达固定常量集合，可作为 Entity、DTO、VO、EO 等结构的字段类型。

### DSL 示例
```json
{
  "name": "room_status_enum",
  "uuid": null,
  "description": "房间状态",
  "moduleName": "room",
  "values": [
    "AVAILABLE",
    "OCCUPIED",
    "DISABLED"
  ]
}
```

### 生成代码
| 代码位置 | 简单描述 |
| --- | --- |
| `RoomStatusEnum` | 枚举定义类，表达常量集合，通常不应手改 |

## 2. 值对象（EO）

### 作用说明
EO 是 DDD 中的值对象，通常用于表达可复用的复合结构，可作为 Entity、DTO、VO、BTO、QTO 的字段类型。

### DSL 示例
```json
{
  "name": "address_eo",
  "description": "地址值对象",
  "uuid": null,
  "moduleName": "user",
  "fieldList": [
    {
      "name": "province",
      "type": "String"
    },
    {
      "name": "city",
      "type": "String"
    },
    {
      "name": "detail_address",
      "type": "String"
    }
  ]
}
```

### 生成代码
| 代码位置 | 简单描述 |
| --- | --- |
| `AddressEo` | EO 结构定义类，用于表达复合值对象 |

## 3. 实体（Entity）

### 作用说明
Entity 对应数据库表，是 ER 建模的核心设计元素，定义字段、主键、索引等数据库结构信息。

### DSL 示例
```text
Entity room {
  name: "room"
  description: "room entity, basic room information"
  primaryFieldName: "room_id"

  fieldList: [
    {
      name: "room_id"
      type: "Long"
      allowNull: false
    },
    {
      name: "room_number"
      type: "String"
      length: 50
      allowNull: false
    },
    {
      name: "room_status"
      type: "Enum"
      typeObject: "room_status_enum"
      allowNull: false
      defaultValue: "AVAILABLE"
    },
    {
      name: "room_type_id"
      type: "Long"
      foreignKey: "room_type"
      foreignKeyType: "many"
      allowNull: false
    }
  ]

  indexList: [
    {
      name: "PRIMARY"
      fieldNameList: ["room_id"]
      unique: true
    },
    {
      name: "IDX_ROOM_NUMBER"
      fieldNameList: ["room_number"]
      unique: true
    }
  ]
}
```

### 生成代码
| 代码位置 | 简单描述 |
| --- | --- |
| `Room` | DO/Entity 结构定义类，对应数据库表 |
| `RoomMapper` | MyBatis-Plus Mapper |
| `RoomDao` | Dao 接口 |
| `RoomDaoImpl` | Dao 实现 |

## 4. 聚合对象（BO）

### 作用说明
BO 用于表达聚合边界，把聚合根实体和子实体组织成树形结构，是写链路和业务不变性校验的基础。

### DSL 示例
知识库未给出完整 Schema，这里用概念 DSL 表示：

```json
{
  "bo": {
    "name": "booking_order_bo",
    "rootEntity": "booking_order",
    "children": [
      {
        "entity": "booking_order_item",
        "fieldName": "booking_order_item_list",
        "relationType": "1:N"
      }
    ]
  }
}
```

### 生成代码
| 代码位置 | 简单描述 |
| --- | --- |
| `BookingOrderBO` | 聚合对象，通常可扩展校验逻辑 |
| `BookingOrderBaseBO` | 聚合对象基类，承载生成的结构与关系代码 |

## 5. 数据传输对象（DTO）

### 作用说明
DTO 用于服务层数据传输，可继承实体字段、通过外键扩展其他 DTO，并可包含少量自定义字段。DTO 不应直接作为 HTTP 返回值。

### DSL 示例
```json
{
  "dto": {
    "uuid": null,
    "name": "room_with_type_dto",
    "description": "房间详情，包含房型",
    "fromEntity": "room",
    "expandList": [
      {
        "foreignKeyInThisEntity": "room_type_id",
        "dtoFieldName": "room_type",
        "dto": {
          "uuid": "room-type-base-dto-uuid",
          "name": "room_type_base_dto",
          "fromEntity": "room_type",
          "description": "房型基础信息"
        }
      }
    ],
    "customFieldList": [
      {
        "name": "available_count",
        "type": "Integer",
        "description": "可用数量"
      }
    ]
  }
}
```

### 生成代码
| 代码位置 | 简单描述 |
| --- | --- |
| `RoomWithTypeDto` | DTO 结构定义 |
| `RoomWithTypeDtoManager` | DTO Manager 接口 |
| `RoomWithTypeDtoManagerImpl` | DTO Manager 实现 |
| `RoomWithTypeDtoConverter` | Entity/BaseDTO 到 DTO 的转换器 |
| `RoomWithTypeDtoService` | DTO 预置查询服务 |
| `RoomWithTypeDtoDataAssembler` | DTO 数据装配与自定义字段扩展点 |
| `RoomWithTypeDtoBaseDataAssembler` | DTO DataAssembler 基类 |

## 6. 视图对象（VO）

### 作用说明
VO 用于 API 返回，通常由 DTO 派生而来，可裁剪字段、扩展 VO 嵌套结构，并补充展示层自定义字段。

### DSL 示例
```json
{
  "vo": {
    "uuid": null,
    "name": "room_list_vo",
    "description": "房间列表展示对象",
    "rootVo": "room_list_vo",
    "moduleName": "room",
    "fromEntity": "room",
    "fromDto": "room_with_type_dto",
    "fromDtoUuid": "room-with-type-dto-uuid",
    "extendFieldList": [
      {
        "name": "room_id"
      },
      {
        "name": "room_number"
      }
    ],
    "expandList": [
      {
        "foreignKeyInThisEntity": "room_type_id",
        "voFieldName": "room_type",
        "vo": {
          "name": "room_type_vo",
          "description": "房型信息",
          "rootVo": "room_list_vo",
          "fromEntity": "room_type",
          "fromDto": "room_type_base_dto",
          "fromDtoUuid": "room-type-base-dto-uuid",
          "extendFieldList": [
            {
              "name": "room_type_id"
            },
            {
              "name": "description"
            }
          ]
        }
      }
    ],
    "customFieldList": []
  }
}
```

### 生成代码
| 代码位置 | 简单描述 |
| --- | --- |
| `RoomListVo` | VO 结构定义，供 API 返回 |
| `RoomListVoConverter` | DTO 到 VO 的结构转换 |
| `RoomListVoDataAssembler` | VO 数据装配与展示扩展点 |
| `RoomListVoBaseDataAssembler` | VO DataAssembler 基类 |

## 7. 读方案（ReadPlan）

### 作用说明
ReadPlan 用于定义复杂查询逻辑，可返回 DTO 或 VO，支持分页、计数、排序和列表字段过滤。

### DSL 示例
```json
{
  "name": "query_available_rooms_excluding_booked",
  "uuid": null,
  "woId": "room-with-booking-wo-uuid",
  "description": "查询可用房间，排除占用时间段",
  "paginationTypes": ["pagination", "count"],
  "query": "room_type.capacity >= #capacityGte AND NOT ( booking_order_item_list contains ( check_in_date < #checkOutDate AND check_out_date > #checkInDate ) )",
  "voOrDtoId": "room-list-vo-uuid",
  "defaultOrder": [
    {
      "fieldPath": "room_id",
      "direction": "ASC"
    }
  ],
  "filters": []
}
```

### 生成代码
| 代码位置 | 简单描述 |
| --- | --- |
| `QueryAvailableRoomsExcludingBookedQto` | 查询参数对象 |
| `QueryAvailableRoomsExcludingBookedQtoDao` | 读方案 Dao |
| `QueryAvailableRoomsExcludingBookedQueryHandler` | 查询入口，返回 DTO 或 VO |

## 8. 写方案（WritePlan）

### 作用说明
WritePlan 是数据库写操作的统一入口，用于在聚合边界内组织新增、更新、删除、批量合并等操作。

### DSL 示例
```json
{
  "uuid": null,
  "name": "create_order_with_items",
  "description": "创建订单及明细",
  "bo": "booking_order",
  "operations": [
    {
      "bo": "booking_order",
      "action": "CREATE",
      "fields": [
        "order_number",
        "member_id",
        "order_time",
        "order_status",
        "total_amount"
      ]
    },
    {
      "bo": "booking_order_item",
      "action": "CREATE",
      "fields": [
        "room_id",
        "check_in_date",
        "check_out_date",
        "actual_amount",
        "item_status"
      ]
    }
  ]
}
```

### 生成代码
| 代码位置 | 简单描述 |
| --- | --- |
| `CreateOrderWithItemsBto` | 写方案参数对象 |
| `BookingOrderBOService` | 写方案对应的服务入口方法 |
| `BaseBookingOrderBOService` | 聚合写入基础实现，负责真正的聚合写操作 |

## 9. 服务层方法（RPC / SERVICE）

### 作用说明
服务层方法分为两类：
- `SERVICE`：模块内可用的服务能力
- `RPC`：对外公开的跨模块能力

它们只在“是否公开”这一点上有区别，方法签名 DSL 结构基本一致。

### DSL 示例
```json
{
  "className": "RoomPriceService",
  "methodName": "calculateTotalPrice",
  "type": "SERVICE",
  "requestParams": [
    {
      "name": "roomTypeId",
      "description": "房型 ID",
      "type": "Long"
    },
    {
      "name": "checkInDate",
      "description": "入住日期",
      "type": "Date"
    },
    {
      "name": "checkOutDate",
      "description": "离店日期",
      "type": "Date"
    }
  ],
  "response": {
    "type": "BigDecimal"
  }
}
```

若改为 RPC，仅需把 `type` 改为 `RPC`。

### 生成代码
| 代码位置 | 简单描述 |
| --- | --- |
| `RoomPriceService` | 服务类及对应方法实现 |
| `PubRoomPriceService` | 当方法为 RPC 且被其他模块订阅时生成 PublicInterface |

## 10. API

### 作用说明
API 用于定义对外 HTTP 接口，负责 URI、请求方式、参数结构和返回结构的契约设计。

### DSL 示例
```json
{
  "methodName": "createMeeting",
  "className": "MeetingManageController",
  "uri": "/api/meeting/create-meeting",
  "requestMethod": "POST",
  "contentType": "JSON",
  "usageScenario": "创建会议",
  "requestParams": [
    {
      "name": "createMeetingBto",
      "description": "创建会议参数",
      "type": "Bto",
      "typeUuid": "create-meeting-write-plan-uuid",
    },
    {
      "name": "test",
      "description": "是否测试请求",
      "type": "Boolean",
    }
  ],
  "response": {
    "type": "Vo",
    "typeUuid": "meeting-detail-vo-uuid"
  }
}
```

### 生成代码
| 代码位置 | 简单描述 |
| --- | --- |
| `MeetingManageController` | Controller 类与 API 方法 |
| `MeetingDetailVo` | API 返回值 VO |
| `${MethodName}Request`（自动内嵌或生成） | POST + JSON 且 body 参数多个时，自动包装请求体 |

## 11. 流程服务（FunctionFlow）

### 作用说明
FunctionFlow 用于表达复杂业务流程，把业务拆成开始节点、顺序节点、条件节点、选择节点，由平台生成流程骨架代码。

### DSL 示例
```json
{
  "name": "user_register",
  "moduleName": "user",
  "uuid": null,
  "description": "注册用户流程",
  "nodes": [
    {
      "name": "start",
      "type": "START_NODE",
      "description": "起始节点"
    },
    {
      "name": "check_user_registered",
      "type": "CONDITION_NODE",
      "description": "检查用户是否已注册"
    },
    {
      "name": "create_user",
      "type": "PROCESS_NODE",
      "description": "创建用户并发送通知"
    },
    {
      "name": "return_user_info",
      "type": "PROCESS_NODE",
      "description": "返回用户信息"
    }
  ],
  "edges": [
    {
      "fromNode": "start",
      "toNode": "check_user_registered"
    },
    {
      "fromNode": "check_user_registered",
      "toNode": "create_user",
      "value": false
    },
    {
      "fromNode": "check_user_registered",
      "toNode": "return_user_info",
      "value": true
    },
    {
      "fromNode": "create_user",
      "toNode": "return_user_info"
    }
  ]
}
```

### 生成代码
| 代码位置 | 简单描述 |
| --- | --- |
| `UserFlowConfig` | 流程注册配置 |
| `UserFlowService` | 流程入口服务 |
| `UserRegisterContext` | 流程上下文对象 |
| `CreateUserNode` | 节点处理类 |

## 12. 领域消息（DomainMessage）

### 作用说明
领域消息用于监听聚合内某个实体的创建、更新、删除事件。消息体由平台自动生成，发送逻辑由平台在实体状态变化时自动触发。

### DSL 示例
```json
{
  "name": "user_created",
  "description": "用户创建成功消息",
  "bo": "user",
  "entity": "user",
  "fields": [
    "user_id",
    "user_name"
  ],
  "action": "create",
  "delayInSeconds": 0,
  "uuid": null
}
```

### 生成代码
| 代码位置 | 简单描述 |
| --- | --- |
| `UserCreatedMo` | 领域消息载体类 |
| Hibernate 监听器生成逻辑 | 在实体变更时自动发送领域消息，不需要手写发送代码 |

## 13. 普通消息（CommonMessage）

### 作用说明
普通消息用于定义一个可主动发送的消息体，不依赖实体状态变化。适合异步通知、跨模块解耦、延迟任务等。

### DSL 示例
```json
{
  "name": "user_registered",
  "description": "用户注册成功通知",
  "uuid": null,
  "moduleName": "user",
  "delay": false,
  "transactional": true,
  "fieldList": [
    {
      "name": "user_id",
      "type": "Long",
      "description": "用户 ID"
    },
    {
      "name": "nick_name",
      "type": "String",
      "description": "昵称"
    }
  ]
}
```

### 生成代码
| 代码位置 | 简单描述 |
| --- | --- |
| `UserRegisteredMo` | 普通消息载体类 |
| `UserRegisteredMoMessageSender` | 消息发送入口类 |

## 14. 订阅消息（SubscribeMessage）

### 作用说明
订阅消息用于把某个领域消息或普通消息订阅到指定模块，平台会自动生成消费模板代码，开发者只需补充消费逻辑。

### DSL 示例
```json
{
  "msgName": "user_registered",
  "moduleName": "coupon"
}
```

也可以通过消息 ID 订阅：

```json
{
  "msgId": "user-registered-msg-uuid",
  "moduleName": "coupon"
}
```

### 生成代码
| 代码位置 | 简单描述 |
| --- | --- |
| `UserRegisteredConsumer` | 消息消费类 |
| `handleMessage(...)` | 消费入口方法，开发者在这里补充业务逻辑 |

## 补充说明

### 1. 不建议直接修改的生成物
以下类型通常应通过修改设计元素后重新生成代码，而不是直接改生成文件：
- Entity / DO
- Enum / EO
- DTO / VO 结构定义
- Mapper / Dao 基础代码
- BaseBO / BaseDataAssembler / BaseQueryHandler 等基类

### 2. 适合手工补充逻辑的扩展点
- `BO` 中的 `validateAggregate` / `valid`
- `DtoDataAssembler#postProcessData`
- `VoDataAssembler#postProcessData`
- `QueryHandler`
- `Controller`
- 消息 `Consumer#handleMessage`
