# MedAI Trinity API 接口文档

<div align="center">

[![API Version](https://img.shields.io/badge/API-v1.0-blue.svg)](https://github.com)
[![Protocol](https://img.shields.io/badge/Protocol-REST-green.svg)](https://github.com)

</div>

---

## 接口概览

MedAI Trinity 提供 RESTful API 接口，支持所有核心功能的调用。

| 接口模块 | 端点前缀 | 说明 | 文档链接 |
|----------|----------|------|----------|
| 病程记录 | `/api/v1/records` | 病程记录生成与管理 | [records.md](./records.md) |
| 质控服务 | `/api/v1/quality` | 病历逻辑质控检查 | [quality.md](./quality.md) |
| DRG服务 | `/api/v1/drg` | DRG分组预测与分析 | [drg.md](./drg.md) |
| 影像分析 | `/api/v1/imaging` | PACS影像智能分析 | [imaging.md](./imaging.md) |
| 系统管理 | `/api/v1/system` | 系统状态与配置管理 | [system.md](./system.md) |

---

## 通用说明

### 基础URL

```
http://localhost:8000/api/v1
```

### 认证方式

所有接口需要在请求头中携带认证信息：

```http
Authorization: Bearer <your_access_token>
```

### 请求格式

```http
Content-Type: application/json
Accept: application/json
```

### 响应格式

所有接口返回统一的JSON格式：

```json
{
  "code": 200,
  "message": "success",
  "data": { ... },
  "timestamp": "2024-03-15T10:30:00Z"
}
```

### 错误码说明

| 错误码 | 说明 |
|--------|------|
| 200 | 成功 |
| 400 | 请求参数错误 |
| 401 | 未授权/认证失败 |
| 403 | 权限不足 |
| 404 | 资源不存在 |
| 429 | 请求频率超限 |
| 500 | 服务器内部错误 |
| 503 | 服务暂时不可用 |

### 分页参数

支持分页的接口使用以下参数：

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| page | integer | 1 | 页码 |
| page_size | integer | 20 | 每页数量（最大100） |

分页响应格式：

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "items": [...],
    "pagination": {
      "page": 1,
      "page_size": 20,
      "total": 100,
      "total_pages": 5
    }
  }
}
```

---

## 快速开始

### 获取访问令牌

```bash
curl -X POST http://localhost:8000/api/v1/auth/token \
  -H "Content-Type: application/json" \
  -d '{
    "username": "admin",
    "password": "your_password"
  }'
```

### 示例请求

```bash
curl -X POST http://localhost:8000/api/v1/records/generate \
  -H "Authorization: Bearer <your_access_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "patient_id": "P001",
    "dialogue_text": "患者主诉头痛3天..."
  }'
```

---

## SDK 示例

### Python

```python
from medai_trinity import MedAIClient

client = MedAIClient(base_url="http://localhost:8000", api_key="your_api_key")

record = client.records.generate(
    patient_id="P001",
    dialogue_text="患者主诉头痛3天..."
)
print(record)
```

### JavaScript

```javascript
const { MedAIClient } = require('medai-trinity-sdk');

const client = new MedAIClient({
  baseUrl: 'http://localhost:8000',
  apiKey: 'your_api_key'
});

const record = await client.records.generate({
  patient_id: 'P001',
  dialogue_text: '患者主诉头痛3天...'
});
console.log(record);
```

---

## 更新日志

| 版本 | 日期 | 更新内容 |
|------|------|----------|
| v1.0.0 | 2024-03-15 | 初始版本发布 |

---

<div align="center">

**[返回主文档](../../README.md)**

</div>
