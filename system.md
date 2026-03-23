# 系统管理接口文档

<div align="center">

[![Module](https://img.shields.io/badge/Module-system-blue.svg)](https://github.com)
[![Version](https://img.shields.io/badge/Version-v1.0-green.svg)](https://github.com)

</div>

---

## 模块概述

系统管理模块提供系统状态监控、配置管理、日志查询和健康检查等功能。

**端点前缀：** `/api/v1/system`

---

## 接口列表

| 方法 | 端点 | 说明 |
|------|------|------|
| GET | `/status` | 获取系统状态 |
| GET | `/health` | 健康检查 |
| GET | `/metrics` | 获取系统指标 |
| GET | `/config` | 获取系统配置 |
| PUT | `/config` | 更新系统配置 |
| GET | `/logs` | 获取系统日志 |
| POST | `/cache/clear` | 清除缓存 |
| GET | `/connections` | 获取连接状态 |

---

## 接口详情

### 1. 获取系统状态

**GET** `/api/v1/system/status`

获取系统整体运行状态。

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "status": "running",
    "version": "1.0.0",
    "build": "20240315001",
    "uptime": "15d 8h 32m 15s",
    "uptime_seconds": 1330335,
    "started_at": "2024-03-01T02:00:00Z",
    "components": {
      "small_model": {
        "status": "healthy",
        "latency": "45ms",
        "requests_total": 125680,
        "requests_success": 125650,
        "error_rate": 0.02
      },
      "large_model": {
        "status": "healthy",
        "latency": "1.2s",
        "requests_total": 8520,
        "requests_success": 8495,
        "error_rate": 0.29,
        "connection": "connected"
      },
      "medclaw": {
        "status": "healthy",
        "connections": {
          "his": "connected",
          "lis": "connected",
          "pacs": "connected",
          "emr": "connected"
        },
        "last_sync": "2024-03-15T10:30:00Z"
      },
      "database": {
        "status": "healthy",
        "type": "sqlite",
        "size": "256MB",
        "connections": 5
      },
      "cache": {
        "status": "healthy",
        "type": "redis",
        "memory_used": "128MB",
        "hit_rate": 95.6
      }
    },
    "resources": {
      "cpu": {
        "usage": 35.2,
        "cores": 4,
        "temperature": 45
      },
      "memory": {
        "total": 8192,
        "used": 5076,
        "free": 3116,
        "usage": 62.0
      },
      "disk": {
        "total": 128000,
        "used": 57600,
        "free": 70400,
        "usage": 45.0
      },
      "network": {
        "bytes_in": 1256000000,
        "bytes_out": 856000000,
        "connections": 15
      }
    },
    "environment": {
      "os": "Raspberry Pi OS",
      "os_version": "Bookworm",
      "python_version": "3.11.2",
      "architecture": "aarch64"
    }
  }
}
```

---

### 2. 健康检查

**GET** `/api/v1/system/health`

用于负载均衡和监控系统的健康检查。

#### 响应示例

**正常状态：**

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "status": "healthy",
    "checks": {
      "database": "pass",
      "cache": "pass",
      "models": "pass",
      "medclaw": "pass"
    },
    "timestamp": "2024-03-15T10:40:00Z"
  }
}
```

**异常状态：**

```json
{
  "code": 503,
  "message": "unhealthy",
  "data": {
    "status": "unhealthy",
    "checks": {
      "database": "pass",
      "cache": "fail",
      "models": "pass",
      "medclaw": "pass"
    },
    "failed_checks": ["cache"],
    "timestamp": "2024-03-15T10:40:00Z"
  }
}
```

---

### 3. 获取系统指标

**GET** `/api/v1/system/metrics`

获取详细的系统性能指标，支持Prometheus格式。

#### 查询参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| format | string | 否 | 输出格式：json/prometheus，默认json |
| period | string | 否 | 时间范围：1h/6h/24h/7d |

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "period": "24h",
    "metrics": {
      "requests": {
        "total": 15680,
        "success": 15620,
        "error": 60,
        "success_rate": 99.62,
        "avg_latency_ms": 125,
        "p50_latency_ms": 85,
        "p95_latency_ms": 350,
        "p99_latency_ms": 850
      },
      "records": {
        "generated": 850,
        "avg_generation_time_s": 45
      },
      "quality_checks": {
        "total": 820,
        "avg_score": 89.5,
        "pass_rate": 95.2
      },
      "drg_predictions": {
        "total": 780,
        "avg_confidence": 0.92
      },
      "imaging_analysis": {
        "total": 125,
        "avg_processing_time_s": 12.5
      }
    },
    "time_series": {
      "requests_per_hour": [
        {"hour": "00:00", "count": 450},
        {"hour": "01:00", "count": 320},
        {"hour": "02:00", "count": 180},
        {"hour": "08:00", "count": 850},
        {"hour": "09:00", "count": 1250},
        {"hour": "10:00", "count": 1580}
      ]
    },
    "collected_at": "2024-03-15T10:45:00Z"
  }
}
```

---

### 4. 获取系统配置

**GET** `/api/v1/system/config`

获取当前系统配置信息。

#### 查询参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| module | string | 否 | 模块名称：medclaw/models/quality/drg/web_ui |

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "config_version": "1.0.0",
    "last_modified": "2024-03-10T08:00:00Z",
    "modules": {
      "medclaw": {
        "server": {
          "host": "0.0.0.0",
          "port": 8080
        },
        "his": {
          "enabled": true,
          "host": "192.168.1.100"
        }
      },
      "models": {
        "small_model": {
          "type": "bitnet",
          "device": "cpu"
        },
        "large_model": {
          "type": "api",
          "enabled": true
        }
      }
    }
  }
}
```

---

### 5. 更新系统配置

**PUT** `/api/v1/system/config`

更新系统配置（需要管理员权限）。

#### 请求参数

```json
{
  "module": "models",
  "config": {
    "small_model": {
      "parameters": {
        "temperature": 0.8,
        "max_tokens": 2048
      }
    }
  },
  "restart_required": false
}
```

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "updated_at": "2024-03-15T11:00:00Z",
    "restart_required": false,
    "changes": [
      {
        "key": "models.small_model.parameters.temperature",
        "old_value": 0.7,
        "new_value": 0.8
      },
      {
        "key": "models.small_model.parameters.max_tokens",
        "old_value": 1024,
        "new_value": 2048
      }
    ]
  }
}
```

---

### 6. 获取系统日志

**GET** `/api/v1/system/logs`

获取系统日志记录。

#### 查询参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| level | string | 否 | 日志级别：debug/info/warning/error |
| module | string | 否 | 模块名称 |
| start_time | string | 否 | 开始时间（ISO 8601） |
| end_time | string | 否 | 结束时间（ISO 8601） |
| keyword | string | 否 | 关键词搜索 |
| page | integer | 否 | 页码 |
| page_size | integer | 否 | 每页数量 |

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "items": [
      {
        "log_id": "LOG20240315001",
        "timestamp": "2024-03-15T10:30:45.123Z",
        "level": "info",
        "module": "records",
        "message": "Record REC20240315001 generated successfully",
        "details": {
          "patient_id": "P001",
          "processing_time": "45s"
        }
      },
      {
        "log_id": "LOG20240315002",
        "timestamp": "2024-03-15T10:31:02.456Z",
        "level": "warning",
        "module": "quality",
        "message": "Quality check found 2 warnings",
        "details": {
          "record_id": "REC20240315001",
          "warning_count": 2
        }
      },
      {
        "log_id": "LOG20240315003",
        "timestamp": "2024-03-15T10:32:15.789Z",
        "level": "error",
        "module": "medclaw",
        "message": "HIS connection timeout",
        "details": {
          "host": "192.168.1.100",
          "timeout": "30s"
        }
      }
    ],
    "pagination": {
      "page": 1,
      "page_size": 50,
      "total": 1250,
      "total_pages": 25
    }
  }
}
```

---

### 7. 清除缓存

**POST** `/api/v1/system/cache/clear`

清除系统缓存（需要管理员权限）。

#### 请求参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| cache_type | string | 否 | 缓存类型：all/model/data/query |
| pattern | string | 否 | 缓存键模式 |

#### 请求示例

```json
{
  "cache_type": "model",
  "pattern": "embedding:*"
}
```

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "cleared_at": "2024-03-15T11:30:00Z",
    "cache_type": "model",
    "keys_cleared": 156,
    "memory_freed": "25.6MB"
  }
}
```

---

### 8. 获取连接状态

**GET** `/api/v1/system/connections`

获取各系统连接状态详情。

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "connections": {
      "his": {
        "status": "connected",
        "host": "192.168.1.100",
        "port": 8080,
        "protocol": "HL7_FHIR",
        "connected_since": "2024-03-01T02:00:00Z",
        "last_activity": "2024-03-15T10:45:00Z",
        "requests_total": 125680,
        "latency_ms": 15
      },
      "lis": {
        "status": "connected",
        "host": "192.168.1.101",
        "port": 8081,
        "protocol": "HL7",
        "connected_since": "2024-03-01T02:00:00Z",
        "last_activity": "2024-03-15T10:44:30Z",
        "requests_total": 45680,
        "latency_ms": 12
      },
      "pacs": {
        "status": "connected",
        "host": "192.168.1.102",
        "port": 11112,
        "protocol": "DICOM",
        "connected_since": "2024-03-01T02:00:00Z",
        "last_activity": "2024-03-15T10:35:00Z",
        "requests_total": 8520,
        "latency_ms": 45
      },
      "emr": {
        "status": "connected",
        "host": "192.168.1.103",
        "port": 8083,
        "protocol": "REST",
        "connected_since": "2024-03-01T02:00:00Z",
        "last_activity": "2024-03-15T10:45:15Z",
        "requests_total": 98520,
        "latency_ms": 18
      },
      "large_model_api": {
        "status": "connected",
        "endpoint": "https://api.medical-llm.com/v1",
        "connected_since": "2024-03-15T08:00:00Z",
        "last_activity": "2024-03-15T10:40:00Z",
        "requests_total": 8520,
        "latency_ms": 1200
      }
    },
    "summary": {
      "total": 5,
      "connected": 5,
      "disconnected": 0
    }
  }
}
```

---

## 系统状态说明

### 状态值

| 状态 | 说明 |
|------|------|
| running | 正常运行 |
| degraded | 降级运行（部分功能不可用） |
| maintenance | 维护模式 |
| stopping | 正在停止 |
| stopped | 已停止 |

### 组件状态

| 状态 | 说明 |
|------|------|
| healthy | 健康 |
| degraded | 降级 |
| unhealthy | 不健康 |
| unknown | 未知 |

---

## 错误码

| 错误码 | 说明 |
|--------|------|
| 40001 | 配置参数无效 |
| 40002 | 模块名称无效 |
| 40301 | 权限不足（需要管理员权限） |
| 40401 | 配置模块不存在 |
| 50001 | 系统内部错误 |
| 50002 | 配置保存失败 |
| 50003 | 缓存清除失败 |

---

<div align="center">

**[返回API目录](./README.md)**

</div>
