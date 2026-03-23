# 病程记录接口文档

<div align="center">

[![Module](https://img.shields.io/badge/Module-records-blue.svg)](https://github.com)
[![Version](https://img.shields.io/badge/Version-v1.0-green.svg)](https://github.com)

</div>

---

## 模块概述

病程记录模块提供病历文书的自动生成、查询、更新和管理功能。

**端点前缀：** `/api/v1/records`

---

## 接口列表

| 方法 | 端点 | 说明 |
|------|------|------|
| POST | `/generate` | 生成病程记录 |
| GET | `/{record_id}` | 获取病程记录详情 |
| PUT | `/{record_id}` | 更新病程记录 |
| DELETE | `/{record_id}` | 删除病程记录 |
| GET | `/list` | 获取病程记录列表 |
| POST | `/export` | 导出病程记录 |

---

## 接口详情

### 1. 生成病程记录

**POST** `/api/v1/records/generate`

根据患者信息和对话内容自动生成合规的首次病程记录。

#### 请求参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| patient_id | string | 是 | 患者ID |
| visit_id | string | 是 | 就诊ID |
| audio_file | string | 否 | Base64编码的音频文件 |
| dialogue_text | string | 否 | 医患对话文本内容 |
| options | object | 否 | 生成选项配置 |

#### options 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| template | string | 否 | 模板类型：first_course_record（首次病程）、progress_note（病程记录）、discharge_summary（出院小结） |
| language | string | 否 | 语言：zh-CN（默认）、en-US |
| include_diagnosis | boolean | 否 | 是否包含初步诊断，默认true |
| include_plan | boolean | 否 | 是否包含诊疗计划，默认true |

#### 请求示例

```json
{
  "patient_id": "P001",
  "visit_id": "V20240315001",
  "audio_file": "UklGRiQAAABXQVZFZm10IBAAAAABAAEARKwAAIhYAQACABAAZGF0YQAAAAA=",
  "dialogue_text": "医生：您好，请问您哪里不舒服？患者：医生，我头痛已经3天了，主要是右侧太阳穴附近跳着疼...",
  "options": {
    "template": "first_course_record",
    "language": "zh-CN",
    "include_diagnosis": true,
    "include_plan": true
  }
}
```

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "record_id": "REC20240315001",
    "patient_id": "P001",
    "visit_id": "V20240315001",
    "content": {
      "chief_complaint": "患者主诉头痛3天",
      "present_illness": "患者3天前无明显诱因出现右侧太阳穴搏动性疼痛，伴恶心、畏光，疼痛程度VAS评分6分，每日发作2-3次，每次持续2-4小时。曾自服布洛芬可缓解。无发热、呕吐、肢体无力等症状。",
      "past_history": "既往体健，否认高血压、糖尿病、冠心病病史。否认肝炎、结核等传染病史。否认手术、外伤史。否认药物、食物过敏史。",
      "personal_history": "生于本地，久居本地，无疫区接触史。吸烟史20年，每日10支；饮酒史15年，每日约100ml白酒。",
      "family_history": "父亲有高血压病史，母亲体健。否认家族遗传病史。",
      "physical_examination": "T: 36.5℃, P: 78次/分, R: 18次/分, BP: 130/85mmHg。神志清楚，精神可，查体合作。双侧瞳孔等大等圆，对光反射灵敏。颈软，无抵抗。心肺腹查体未见明显异常。四肢肌力、肌张力正常，病理征阴性。",
      "auxiliary_examination": "血常规：WBC 6.5×10^9/L, HGB 135g/L, PLT 210×10^9/L。头颅CT：未见明显异常。",
      "preliminary_diagnosis": "1. 偏头痛\n2. 高血压病（待排）",
      "diagnosis_plan": "1. 完善血压监测、血脂、血糖等检查\n2. 给予止痛、改善循环等对症治疗\n3. 必要时完善头颅MRI检查\n4. 神经内科随诊"
    },
    "generated_at": "2024-03-15T10:30:00Z",
    "processing_time": "45s",
    "model_used": "bitnet-medical-small"
  },
  "timestamp": "2024-03-15T10:30:45Z"
}
```

---

### 2. 获取病程记录详情

**GET** `/api/v1/records/{record_id}`

根据记录ID获取病程记录详情。

#### 路径参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| record_id | string | 是 | 病程记录ID |

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "record_id": "REC20240315001",
    "patient_id": "P001",
    "visit_id": "V20240315001",
    "content": {
      "chief_complaint": "患者主诉头痛3天",
      "present_illness": "患者3天前无明显诱因出现..."
    },
    "status": "draft",
    "created_at": "2024-03-15T10:30:00Z",
    "updated_at": "2024-03-15T10:35:00Z",
    "created_by": "doctor_001"
  }
}
```

---

### 3. 更新病程记录

**PUT** `/api/v1/records/{record_id}`

更新已有的病程记录内容。

#### 路径参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| record_id | string | 是 | 病程记录ID |

#### 请求参数

```json
{
  "content": {
    "chief_complaint": "患者主诉头痛3天，加重1天",
    "present_illness": "患者3天前无明显诱因出现右侧太阳穴搏动性疼痛..."
  },
  "status": "finalized"
}
```

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "record_id": "REC20240315001",
    "updated_at": "2024-03-15T11:00:00Z",
    "version": 2
  }
}
```

---

### 4. 删除病程记录

**DELETE** `/api/v1/records/{record_id}`

删除指定的病程记录（软删除）。

#### 路径参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| record_id | string | 是 | 病程记录ID |

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "record_id": "REC20240315001",
    "deleted_at": "2024-03-15T12:00:00Z"
  }
}
```

---

### 5. 获取病程记录列表

**GET** `/api/v1/records/list`

分页获取病程记录列表。

#### 查询参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| patient_id | string | 否 | 患者ID筛选 |
| visit_id | string | 否 | 就诊ID筛选 |
| status | string | 否 | 状态筛选：draft/finalized/archived |
| start_date | string | 否 | 开始日期（YYYY-MM-DD） |
| end_date | string | 否 | 结束日期（YYYY-MM-DD） |
| page | integer | 否 | 页码，默认1 |
| page_size | integer | 否 | 每页数量，默认20 |

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "items": [
      {
        "record_id": "REC20240315001",
        "patient_id": "P001",
        "chief_complaint": "患者主诉头痛3天",
        "status": "finalized",
        "created_at": "2024-03-15T10:30:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "page_size": 20,
      "total": 45,
      "total_pages": 3
    }
  }
}
```

---

### 6. 导出病程记录

**POST** `/api/v1/records/export`

将病程记录导出为指定格式。

#### 请求参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| record_ids | array | 是 | 记录ID列表 |
| format | string | 是 | 导出格式：pdf/docx/html |
| include_signature | boolean | 否 | 是否包含签名区域 |

#### 请求示例

```json
{
  "record_ids": ["REC20240315001", "REC20240315002"],
  "format": "pdf",
  "include_signature": true
}
```

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "export_id": "EXP20240315001",
    "format": "pdf",
    "download_url": "/api/v1/downloads/EXP20240315001",
    "expires_at": "2024-03-15T12:00:00Z"
  }
}
```

---

## 错误码

| 错误码 | 说明 |
|--------|------|
| 40001 | 患者ID不存在 |
| 40002 | 就诊ID不存在 |
| 40003 | 音频文件格式不支持 |
| 40004 | 对话内容为空 |
| 40005 | 模板类型无效 |
| 40401 | 病程记录不存在 |
| 50001 | 模型生成失败 |

---

<div align="center">

**[返回API目录](./README.md)**

</div>
