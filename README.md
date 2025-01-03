# 杏桃 - AI中间件统一处理执行

> 该项目处于早起计划版本（画大饼），如果你有兴趣，欢迎参与。
> 
> 如果你有更好的项目也欢迎向我们推荐！

## 项目目标

1. 建立一个以中心AI为核心的中间件，统一管理聊天插件与API服务的交互。


2. 提供智能化的处理功能，使用户可以通过自然语言执行复杂操作。


3. 规范各插件与API服务的对接方式，提高管理与执行效率。


---

## 系统架构设计

### 中心AI处理核心

负责消息解析、自然语言处理、指令分发、逻辑协调。

转发指令到注册的插件或API服务。

支持复杂指令解析（如一句话执行多条指令）。


### 聊天插件层

各类聊天插件（如QQ机器人插件等，服务器侧提供功能的）与中心AI交互。

注册支持的指令及介绍。

定义特定情况下可绕过中心AI直接处理的规则。


### API服务层

不直接与聊天插件交互。

通过中心AI处理消息或指令并转发请求（机器人侧）。


---

## 核心功能模块

### 1. 指令注册机制

注册内容：

插件名称和版本。

插件支持的指令列表和描述（如用途、参数）。

需要绕过中心AI处理的特殊情况定义。


注册流程：

每个聊天插件在初始化时，向中心AI发送一份指令注册表。

中心AI将指令注册表存储并生成唯一标识。



### 2. 消息处理流程

1. 聊天插件接收到用户消息后，转发到中心AI。


2. 中心AI分析消息：

若消息符合注册指令，执行对应操作。

若符合绕过规则，直接转发到插件处理。

若为复杂操作（如多条指令），分解后逐一处理并返回结果。



3. 处理完成后，中心AI将结果返回给聊天插件，或调用API服务并反馈执行情况。



### 3. API转发机制

聊天插件需注册所需的API服务接口。

中心AI负责验证插件的API调用权限，防止越权。

API请求需附带调用指令的上下文信息，便于API进行智能化处理。

---

## 设计规范

### 1. 聊天插件开发要求

**功能规范**

必须向中心AI注册指令列表，未注册指令不能处理。

明确绕过中心AI处理的场景（如特定敏感指令）。


**通信协议**

统一使用标准协议（如HTTP、WebSocket）与中心AI通信。

必须实现标准化的注册接口。



### 2. 中心AI处理要求

**自然语言解析**

支持多语言识别，能高效解析自然语言的复杂指令。

具备学习和优化能力，持续改进解析准确率。


**指令分发**

优先匹配注册的插件指令。

若未匹配，调用默认逻辑或返回错误提示。



### 3. API服务开发要求

**功能规范**

必须由中心AI中转，不允许与聊天插件直接通信。

需定义可被调用的功能及参数规则。


**权限管理**

所有API请求需通过权限校验。

API接口需记录调用日志，确保可追踪性。


---


## 消息格式

以下是 **聊天插件** 和 **API插件** 的注册数据格式，设计目标是确保插件能够清晰地向中心AI声明其功能、指令处理能力和转发规则。  

### **1. 聊天插件注册格式**  
```
{
  "plugin_id": "string",                  // 聊天插件的唯一标识符
  "plugin_name": "string",                // 聊天插件名称
  "version": "string",                    // 聊天插件版本号
  "supported_commands": [                 // 聊天插件支持的指令列表
    {
      "command": "string",                // 指令名称
      "description": "string",            // 指令功能描述
      "requires_processing": "boolean"    // 是否需要中心AI预处理（true/false）
    }
  ],
  "default_handling_policy": {            // 默认处理策略（当消息未匹配任何注册指令时）
    "forward_to_central_ai": "boolean",   // 未匹配时是否转发至中心AI
    "handle_by_self": "boolean"           // 未匹配时是否直接由插件处理
  }
}
```

#### **字段说明**  
| 字段名                  | 类型       | 描述                                                                                     |  
|-------------------------|------------|------------------------------------------------------------------------------------------|  
| `plugin_id`             | `string`   | 聊天插件的唯一标识符，用于标识该插件（例如UUID）。                                        |  
| `plugin_name`           | `string`   | 聊天插件名称，例如 "QQBot" 或 "TelegramBot"。                                             |  
| `version`               | `string`   | 聊天插件版本号，用于版本管理。                                                           |  
| `supported_commands`    | `array`    | 聊天插件支持的指令及其描述。                                                             |  
| `command`               | `string`   | 指令名称，例如 "query_player" 或 "kick_player"。                                          |  
| `description`           | `string`   | 指令的功能简述，例如 "查询在线玩家" 或 "踢出指定玩家"。                                    |  
| `requires_processing`   | `boolean`  | 指令是否需要由中心AI预处理，`true` 表示需要经过自然语言解析等处理。                        |  
| `default_handling_policy` | `object` | 当消息未匹配注册指令时的处理方式，是否转发给中心AI或由插件自行处理。                      |  

---

### **2. API插件注册格式**  
```
{
  "api_id": "string",                     // API插件的唯一标识符
  "api_name": "string",                   // API插件名称
  "version": "string",                    // API插件版本号
  "supported_endpoints": [                // API插件支持的接口列表
    {
      "endpoint": "string",               // API接口名称
      "description": "string",            // 接口功能描述
      "params_schema": {                  // 接口的参数结构定义
        "param_name": "param_type"        // 参数名称和类型（如 "player_name": "string"）
      }
    }
  ],
  "default_policy": {                     // 默认处理策略
    "allow_direct_access": "boolean",     // 是否允许聊天插件直接调用API（通常为false，必须由中心AI转发）
    "require_auth": "boolean"             // 是否需要认证（如访问令牌）才能调用该API
  }
}
```

#### **字段说明**  
| 字段名                  | 类型       | 描述                                                                                     |  
|-------------------------|------------|------------------------------------------------------------------------------------------|  
| `api_id`                | `string`   | API插件的唯一标识符，用于标识该插件（例如UUID）。                                         |  
| `api_name`              | `string`   | API插件名称，例如 "ServerManager" 或 "GameDataAPI"。                                     |  
| `version`               | `string`   | API插件版本号，用于版本管理。                                                            |  
| `supported_endpoints`   | `array`    | API插件支持的接口及其描述。                                                              |  
| `endpoint`              | `string`   | API接口名称，例如 "get_online_players" 或 "ban_player"。                                 |  
| `description`           | `string`   | 接口功能描述，例如 "获取在线玩家列表" 或 "封禁玩家"。                                     |  
| `params_schema`         | `object`   | API接口参数结构定义，列出参数名称及其类型（如 "string", "integer", "boolean"）。          |  
| `default_policy`        | `object`   | 默认处理策略，规定是否允许直接调用API及是否需要认证。                                    |  

---

### **注册示例**  

#### **聊天插件注册示例**  
```json
{
  "plugin_id": "plugin_qqbot_001",
  "plugin_name": "QQBot",
  "version": "1.0.0",
  "supported_commands": [
    {
      "command": "query_player",
      "description": "查询在线玩家",
      "requires_processing": true
    },
    {
      "command": "kick_player",
      "description": "踢出指定玩家",
      "requires_processing": true
    }
  ],
  "default_handling_policy": {
    "forward_to_central_ai": true,
    "handle_by_self": false
  }
}
```

#### **API插件注册示例**  
```json
{
  "api_id": "api_server_manager_001",
  "api_name": "ServerManagerPlugin",
  "version": "1.0.0",
  "supported_endpoints": [
    {
      "endpoint": "get_online_players",
      "description": "获取在线玩家列表",
      "params_schema": {}
    },
    {
      "endpoint": "kick_player",
      "description": "踢出指定玩家",
      "params_schema": {
        "player_name": "string"
      }
    }
  ],
  "default_policy": {
    "allow_direct_access": false,
    "require_auth": true
  }
}
```


---

### **3. 通信格式**  

```
{
  "message_id": "string",           // 消息的唯一标识符（UUID格式）
  "timestamp": "string",            // 消息发送的时间戳（ISO 8601格式，如2025-01-02T10:00:00Z）
  "source": {                       // 消息来源信息
    "type": "string",               // 来源类型（如 "api", "plugin", 或 "ai"）
    "name": "string",               // 来源名称（如 "QQBot", "CentralAI"）
    "version": "string"             // 来源版本号（如 "1.0.0"）
  },
  "target": {                       // 消息目标信息
    "type": "string",               // 目标类型（如 "plugin", "api", 或 "ai"）
    "name": "string"                // 目标名称（如 "ServerManagerPlugin"）
  },
  "content": {                      // 消息内容
    "type": "string",               // 消息类型（如 "raw_message", "command", "response", "error"）
    "data": {                       // 消息数据，根据类型不同格式有所变化
      "raw_message": {              // 原始消息，仅适用于 type = "raw_message"
        "message_text": "string",   // 用户的原文消息
        "sender": {                 // 发送者信息
          "user_id": "string",      // 发送者的用户ID
          "nickname": "string"      // 发送者昵称
        },
        "context": {                // 上下文信息
          "chat_id": "string",      // 聊天群ID或私聊ID
          "message_type": "string"  // 消息类型（如 "group", "private"）
        }
      },
      "command": {                  // 指令消息，仅适用于 type = "command"
        "command": "string",        // 指令名称
        "params": {                 // 指令参数（键值对形式）
          "key": "value"
        }
      },
      "response": {                 // 响应消息，仅适用于 type = "response"
        "status": "string",         // 执行状态（如 "success", "failure"）
        "result": "string"          // 执行结果或返回值
      },
      "error": {                    // 错误消息，仅适用于 type = "error"
        "code": "string",           // 错误码
        "message": "string"         // 错误描述
      }
    }
  },
  "context": {                      // 消息的上下文信息
    "session_id": "string",         // 会话ID，用于标识同一交互的上下文
    "parent_message_id": "string",  // 父级消息ID（如果有，标识消息链路）
    "trace": [                      // 消息流转路径（按顺序记录每一步）
      {
        "timestamp": "string",      // 时间戳
        "handler": "string"         // 处理节点名称（如 "CentralAI", "ServerManagerPlugin"）
      }
    ]
  }
}
```

### **字段描述**

#### **顶层字段**

| 字段         | 类型     | 描述                                         |
| ------------ | -------- | -------------------------------------------- |
| `message_id` | `string` | 唯一标识此消息的ID，用于追踪消息流转。       |
| `timestamp`  | `string` | 消息生成或发送的时间戳，格式为ISO 8601标准。 |
| `source`     | `object` | 消息来源信息，说明消息的发出方。             |
| `target`     | `object` | 消息目标信息，说明消息的接收方。             |
| `content`    | `object` | 消息内容，包含消息的实际数据和类型信息。     |
| `context`    | `object` | 上下文信息，提供消息的会话和链路追踪能力。   |

------

#### **`source`和`target`字段**

| 字段      | 类型     | 描述                                                  |
| --------- | -------- | ----------------------------------------------------- |
| `type`    | `string` | 来源或目标的类型，如 "api", "plugin", "ai"。          |
| `name`    | `string` | 来源或目标的名称，如 "QQBot", "ServerManagerPlugin"。 |
| `version` | `string` | 来源或目标的版本号，如 "1.0.0"。                      |

------

#### **`content`字段**

| 字段   | 类型     | 描述                                                         |
| ------ | -------- | ------------------------------------------------------------ |
| `type` | `string` | 消息的类型，可选值： "raw_message", "command", "response", "error"。 |
| `data` | `object` | 消息的具体数据，不同类型有不同结构。                         |

------

#### **`content.data`字段按类型描述**

- **`type = "raw_message"`**

  | 字段           | 类型     | 描述                             |
  | -------------- | -------- | -------------------------------- |
  | `message_text` | `string` | 用户的原始消息内容。             |
  | `sender`       | `object` | 发送者信息，包括用户ID和昵称。   |
  | `context`      | `object` | 聊天上下文信息，如群ID或私聊ID。 |

- **`type = "command"`**

  | 字段      | 类型     | 描述                   |
  | --------- | -------- | ---------------------- |
  | `command` | `string` | 指令名称。             |
  | `params`  | `object` | 指令参数，键值对形式。 |

- **`type = "response"`**

  | 字段     | 类型     | 描述                                |
  | -------- | -------- | ----------------------------------- |
  | `status` | `string` | 执行状态，如 "success", "failure"。 |
  | `result` | `string` | 执行结果或返回数据。                |

- **`type = "error"`**

  | 字段      | 类型     | 描述                         |
  | --------- | -------- | ---------------------------- |
  | `code`    | `string` | 错误代码，用于标识错误类型。 |
  | `message` | `string` | 错误描述，说明错误原因。     |

------

#### **`context`字段**

| 字段                | 类型     | 描述                                   |
| ------------------- | -------- | -------------------------------------- |
| `session_id`        | `string` | 会话ID，用于标识一组关联消息。         |
| `parent_message_id` | `string` | 父级消息ID，用于追踪消息链路。         |
| `trace`             | `array`  | 消息流转路径，记录每个处理节点的信息。 |

------


## 示例流程

具体消息示例：查询服务器在线玩家并踢出玩家abc

以下是完整的消息交互流程：


### 1. QQ管理员向机器人发送消息

管理员消息内容：

> 查询服务器在线玩家数量，并把叫abc的玩家踢出服务器。

机器人转发消息到中心AI：

```json
{
  "message_id": "123e4567-e89b-12d3-a456-426614174000",
  "timestamp": "2025-01-02T10:00:00Z",
  "source": {
    "type": "api",
    "name": "QQBot",
    "version": "1.0.0"
  },
  "target": {
    "type": "ai",
    "name": "CentralAI"
  },
  "content": {
    "type": "raw_message",
    "data": {
      "message_text": "查询服务器在线玩家数量，并把叫abc的玩家踢出服务器。",
      "sender": {
        "user_id": "admin_001",
        "nickname": "管理员"
      },
      "context": {
        "chat_id": "group_12345", // QQ群ID
        "message_type": "group"   // 消息类型（私聊或群聊）
      }
    }
  },
  "context": {
    "session_id": "abc123",
    "parent_message_id": null
  }
}
```

### 2. 中心AI解析消息并生成指令

中心AI解析自然语言后，将其拆分为两条指令：

1. 查询服务器在线玩家数量。

2. 踢出名为abc的玩家。


中心AI生成的第一条指令：查询在线玩家

```json
{
  "message_id": "123e4567-e89b-12d3-a456-426614174001",
  "timestamp": "2025-01-02T10:00:01Z",
  "source": {
    "type": "ai",
    "name": "CentralAI",
    "version": "2.0.0"
  },
  "target": {
    "type": "plugin",
    "name": "ServerManagerPlugin"
  },
  "content": {
    "type": "command",
    "data": {
      "command": "get_online_players",
      "params": {
        "server_id": "mc_001"
      }
    }
  },
  "context": {
    "session_id": "abc123",
    "parent_message_id": "123e4567-e89b-12d3-a456-426614174000"
  }
}
```

中心AI生成的第二条指令：踢出玩家abc

```json
{
  "message_id": "123e4567-e89b-12d3-a456-426614174002",
  "timestamp": "2025-01-02T10:00:02Z",
  "source": {
    "type": "ai",
    "name": "CentralAI",
    "version": "2.0.0"
  },
  "target": {
    "type": "plugin",
    "name": "ServerManagerPlugin"
  },
  "content": {
    "type": "command",
    "data": {
      "command": "kick_player",
      "params": {
        "server_id": "mc_001",
        "player_name": "abc"
      }
    }
  },
  "context": {
    "session_id": "abc123",
    "parent_message_id": "123e4567-e89b-12d3-a456-426614174000"
  }
}
```

### 3. 聊天插件执行指令并返回结果

第一条指令的返回结果（在线玩家数量）：

```json
{
  "message_id": "123e4567-e89b-12d3-a456-426614174003",
  "timestamp": "2025-01-02T10:00:03Z",
  "source": {
    "type": "plugin",
    "name": "ServerManagerPlugin",
    "version": "1.0.0"
  },
  "target": {
    "type": "ai",
    "name": "CentralAI"
  },
  "content": {
    "type": "response",
    "data": {
      "response": {
        "status": "success",
        "result": {
          "online_players": ["abc", "player2", "player3"]
        }
      }
    }
  },
  "context": {
    "session_id": "abc123",
    "parent_message_id": "123e4567-e89b-12d3-a456-426614174001"
  }
}
```

第二条指令的返回结果（踢出玩家abc）：

```json
{
  "message_id": "123e4567-e89b-12d3-a456-426614174004",
  "timestamp": "2025-01-02T10:00:04Z",
  "source": {
    "type": "plugin",
    "name": "ServerManagerPlugin",
    "version": "1.0.0"
  },
  "target": {
    "type": "ai",
    "name": "CentralAI"
  },
  "content": {
    "type": "response",
    "data": {
      "response": {
        "status": "success",
        "result": "Player abc has been kicked."
      }
    }
  },
  "context": {
    "session_id": "abc123",
    "parent_message_id": "123e4567-e89b-12d3-a456-426614174002"
  }
}
```

### 4. 中心AI整合结果并返回机器人

中心AI整合两条结果后，发送给QQ机器人：

```json
{
  "message_id": "123e4567-e89b-12d3-a456-426614174005",
  "timestamp": "2025-01-02T10:00:05Z",
  "source": {
    "type": "ai",
    "name": "CentralAI",
    "version": "2.0.0"
  },
  "target": {
    "type": "api",
    "name": "QQBot"
  },
  "content": {
    "type": "response",
    "data": {
      "response": {
        "status": "success",
        "result": "当前在线玩家数量为3：abc, player2, player3。\n玩家abc已被踢出服务器。"
      }
    }
  },
  "context": {
    "session_id": "abc123",
    "parent_message_id": "123e4567-e89b-12d3-a456-426614174000"
  }
}
```

### 5. QQ机器人向管理员反馈

反馈内容：

> 当前在线玩家数量为3：abc, player2, player3。
玩家abc已被踢出服务器。

---

原文转发：机器人直接将管理员的原文消息传递至中心AI。

自然语言处理：中心AI解析消息内容并拆分为两条指令。

指令执行与反馈：中心AI协调插件完成任务，并将整合结果返回给机器人。

这种流程确保了逻辑清晰、扩展性强，并能智能处理复杂指令。


---

## 项目管理建议

### 1. 模块化开发

聊天插件、中心AI、API服务独立开发，遵循统一接口规范。



### 2. 可扩展性

支持动态注册新插件和API服务，无需重启系统。



### 3. 日志与监控

记录所有交互日志，提供实时监控和报表功能。



### 4. 安全性

完整的权限校验和对应的处理，通过代码和AI双重校验



---

此设计旨在提升系统整体效率与灵活性，满足用户快速响应复杂需求的目标。
