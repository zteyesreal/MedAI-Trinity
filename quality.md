# 质控服务接口文档

<div align="center">

[![Module](https://img.shields.io/badge/Module-quality-blue.svg)](https://github.com)
[![Version](https://img.shields.io/badge/Version-v1.0-green.svg)](https://github.com)

</div>

---

## 模块概述

质控服务模块提供病历全维度逻辑质控检查功能，包括时序一致性、必填项完整性、药物相互作用、诊断匹配等检查。

**端点前缀：** `/api/v1/quality`

---

## 接口列表

| 方法 | 端点 | 说明 |
|------|------|------|
| POST | `/check` | 执行质控检查 |
| GET | `/{check_id}` | 获取质控结果详情 |
| GET | `/rules` | 获取质控规则列表 |
| PUT | `/rules/{rule_id}` | 更新质控规则 |
| POST | `/batch` | 批量质控检查 |

---

## 接口详情

### 1. 执行质控检查

**POST** `/api/v1/quality/check`

对病历进行全维度逻辑质控检查。

#### 请求参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| record_id | string | 是 | 病程记录ID |
| record_content | object | 是 | 病历内容对象 |
| check_types | array | 否 | 检查类型列表 |
| strict_mode | boolean | 否 | 严格模式，默认false |

#### check_types 可选值

| 值 | 说明 |
|------|------|
| temporal | 时序逻辑一致性检查 |
| completeness | 必填项完整性检查 |
| medication | 药物相互作用检查 |
| diagnosis | 诊断与检查匹配检查 |
| icd | ICD编码规范性检查 |
| all | 执行所有检查（默认） |

#### 请求示例

```json
{
  "record_id": "REC20240315001",
  "record_content": {
    "chief_complaint": "患者主诉头痛3天",
    "present_illness": "患者3天前无明显诱因出现右侧太阳穴搏动性疼痛...",
    "past_history": "既往体健，否认高血压、糖尿病病史",
    "allergy_history": "",
    "physical_examination": {
      "temperature": "36.5",
      "pulse": "78",
      "respiration": "18",
      "blood_pressure": "130/85"
    },
    "medications": [
      {
        "name": "阿司匹林肠溶片",
        "dosage": "100mg",
        "frequency": "qd",
        "route": "po"
      },
      {
        "name": "布洛芬缓释胶囊",
        "dosage": "300mg",
        "frequency": "prn",
        "route": "po"
      }
    ],
    "diagnoses": [
      {"code": "G43.9", "name": "偏头痛"},
      {"code": "I10", "name": "原发性高血压"}
    ]
  },
  "check_types": ["temporal", "completeness", "medication", "diagnosis"],
  "strict_mode": false
}
```

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "check_id": "QC20240315001",
    "record_id": "REC20240315001",
    "overall_score": 92,
    "passed": true,
    "summary": {
      "total_checks": 15,
      "passed_checks": 13,
      "warning_count": 2,
      "error_count": 0,
      "info_count": 3
    },
    "issues": [
      {
        "id": "ISS001",
        "type": "warning",
        "category": "completeness",
        "rule_id": "QC002",
        "rule_name": "required_fields",
        "field": "allergy_history",
        "field_display": "过敏史",
        "message": "过敏史未填写",
        "suggestion": "请补充患者过敏史信息，若患者无过敏史请填写"否认"",
        "severity_score": -5,
        "auto_fixable": true,
        "auto_fix_value": "否认药物、食物过敏史"
      },
      {
        "id": "ISS002",
        "type": "info",
        "category": "medication",
        "rule_id": "QC003",
        "rule_name": "medication_interaction",
        "field": "medications[0]",
        "field_display": "阿司匹林肠溶片",
        "message": "阿司匹林建议餐后服用",
        "suggestion": "可考虑添加服药时间说明，如"餐后30分钟服用"",
        "severity_score": 0,
        "auto_fixable": false
      },
      {
        "id": "ISS003",
        "type": "info",
        "category": "diagnosis",
        "rule_id": "QC004",
        "rule_name": "diagnosis_match",
        "field": "diagnoses[1]",
        "field_display": "原发性高血压",
        "message": "诊断"原发性高血压"缺少相关检查支持",
        "suggestion": "建议完善血压监测或添加高血压相关检查结果",
        "severity_score": 0,
        "auto_fixable": false
      }
    ],
    "check_details": {
      "temporal": {
        "status": "passed",
        "score": 100,
        "details": "时间顺序逻辑正确"
      },
      "completeness": {
        "status": "warning",
        "score": 90,
        "missing_fields": ["allergy_history"]
      },
      "medication": {
        "status": "passed",
        "score": 95,
        "interactions": [],
        "suggestions": 1
      },
      "diagnosis": {
        "status": "passed",
        "score": 88,
        "mismatches": 0,
        "unsupported": 1
      }
    },
    "checked_at": "2024-03-15T10:31:00Z",
    "processing_time": "2.3s"
  }
}
```

---

### 2. 获取质控结果详情

**GET** `/api/v1/quality/{check_id}`

根据检查ID获取质控结果详情。

#### 路径参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| check_id | string | 是 | 质控检查ID |

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "check_id": "QC20240315001",
    "record_id": "REC20240315001",
    "overall_score": 92,
    "passed": true,
    "issues": [...],
    "checked_at": "2024-03-15T10:31:00Z"
  }
}
```

---

### 3. 获取质控规则列表

**GET** `/api/v1/quality/rules`

获取所有可用的质控规则。

#### 查询参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| category | string | 否 | 规则分类筛选 |
| enabled | boolean | 否 | 是否只返回启用的规则 |
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
        "rule_id": "QC001",
        "name": "temporal_consistency",
        "display_name": "时序逻辑一致性检查",
        "category": "temporal",
        "enabled": true,
        "severity": "error",
        "description": "检查病历中时间顺序的逻辑一致性",
        "parameters": {
          "check_admission_time": true,
          "check_discharge_time": true,
          "check_medication_time": true
        }
      },
      {
        "rule_id": "QC002",
        "name": "required_fields",
        "display_name": "必填项检查",
        "category": "completeness",
        "enabled": true,
        "severity": "warning",
        "description": "检查病历必填字段是否完整"
      },
      {
        "rule_id": "QC003",
        "name": "medication_interaction",
        "display_name": "药物相互作用检查",
        "category": "medication",
        "enabled": true,
        "severity": "error",
        "description": "检查处方药物之间的相互作用"
      }
    ],
    "pagination": {
      "page": 1,
      "page_size": 20,
      "total": 5,
      "total_pages": 1
    }
  }
}
```

---

### 4. 更新质控规则

**PUT** `/api/v1/quality/rules/{rule_id}`

更新指定质控规则的配置。

#### 路径参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| rule_id | string | 是 | 规则ID |

#### 请求参数

```json
{
  "enabled": true,
  "severity": "warning",
  "parameters": {
    "check_admission_time": true,
    "check_discharge_time": true,
    "check_medication_time": false
  }
}
```

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "rule_id": "QC001",
    "updated_at": "2024-03-15T11:00:00Z"
  }
}
```

---

### 5. 批量质控检查

**POST** `/api/v1/quality/batch`

对多条病历进行批量质控检查。

#### 请求参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| record_ids | array | 是 | 病程记录ID列表 |
| check_types | array | 否 | 检查类型列表 |

#### 请求示例

```json
{
  "record_ids": ["REC20240315001", "REC20240315002", "REC20240315003"],
  "check_types": ["completeness", "medication"]
}
```

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "batch_id": "BATCH20240315001",
    "total": 3,
    "completed": 3,
    "results": [
      {
        "record_id": "REC20240315001",
        "check_id": "QC20240315001",
        "passed": true,
        "score": 92
      },
      {
        "record_id": "REC20240315002",
        "check_id": "QC20240315002",
        "passed": false,
        "score": 75
      },
      {
        "record_id": "REC20240315003",
        "check_id": "QC20240315003",
        "passed": true,
        "score": 98
      }
    ],
    "statistics": {
      "average_score": 88.3,
      "pass_rate": 66.7
    }
  }
}
```

---

## 质控规则说明

### 时序逻辑一致性检查 (temporal_consistency)

检查病历中各时间节点的逻辑关系：
- 入院时间 ≤ 首次病程记录时间
- 首次病程记录时间 ≤ 日常病程记录时间
- 日常病程记录时间 ≤ 出院时间
- 用药时间在住院时间范围内

### 必填项完整性检查 (required_fields)

检查病历必填字段是否完整：
- 主诉 (chief_complaint)
- 现病史 (present_illness)
- 既往史 (past_history)
- 体格检查 (physical_examination)
- 初步诊断 (preliminary_diagnosis)

### 药物相互作用检查 (medication_interaction)

检查处方药物相关问题：
- 药物相互作用
- 重复用药
- 禁忌症
- 剂量合理性

### 诊断与检查匹配检查 (diagnosis_match)

检查诊断与检查结果的一致性：
- 诊断是否有检查结果支持
- 检查结果异常是否有相应诊断

---

## 错误码

| 错误码 | 说明 |
|--------|------|
| 40001 | 病程记录ID不存在 |
| 40002 | 病历内容为空 |
| 40003 | 检查类型无效 |
| 40401 | 质控结果不存在 |
| 40402 | 质控规则不存在 |
| 50001 | 质控引擎执行失败 |

---

<div align="center">

**[返回API目录](./README.md)**

</div>
