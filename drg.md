# DRG服务接口文档

<div align="center">

[![Module](https://img.shields.io/badge/Module-drg-blue.svg)](https://github.com)
[![Version](https://img.shields.io/badge/Version-v1.0-green.svg)](https://github.com)

</div>

---

## 模块概述

DRG服务模块提供DRG分组预测、费用分析和医保预警功能，帮助医院优化医保结算和成本控制。

**端点前缀：** `/api/v1/drg`

---

## 接口列表

| 方法 | 端点 | 说明 |
|------|------|------|
| POST | `/predict` | DRG分组预测 |
| GET | `/{prediction_id}` | 获取预测结果详情 |
| POST | `/analyze` | 费用分析 |
| GET | `/groups` | 获取DRG分组列表 |
| GET | `/rules` | 获取分组规则 |

---

## 接口详情

### 1. DRG分组预测

**POST** `/api/v1/drg/predict`

根据诊断、手术和患者信息预测DRG分组。

#### 请求参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| patient_id | string | 是 | 患者ID |
| visit_id | string | 是 | 就诊ID |
| diagnoses | array | 是 | 诊断列表 |
| procedures | array | 否 | 手术/操作列表 |
| patient_info | object | 是 | 患者基本信息 |
| options | object | 否 | 预测选项 |

#### diagnoses 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| code | string | 是 | ICD-10诊断编码 |
| name | string | 是 | 诊断名称 |
| type | string | 否 | 诊断类型：primary（主诊断）、secondary（次要诊断） |

#### procedures 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| code | string | 是 | ICD-9-CM-3手术编码 |
| name | string | 是 | 手术/操作名称 |
| date | string | 否 | 手术日期 |

#### patient_info 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| age | integer | 是 | 年龄 |
| gender | string | 是 | 性别：M/F |
| length_of_stay | integer | 否 | 住院天数 |
| weight | float | 否 | 体重（kg） |
| neonatal_age | integer | 否 | 新生儿日龄 |

#### 请求示例

```json
{
  "patient_id": "P001",
  "visit_id": "V20240315001",
  "diagnoses": [
    {"code": "I10", "name": "原发性高血压", "type": "primary"},
    {"code": "E11.9", "name": "2型糖尿病", "type": "secondary"},
    {"code": "I25.1", "name": "动脉粥样硬化性心脏病", "type": "secondary"}
  ],
  "procedures": [
    {"code": "00.66", "name": "经皮冠状动脉球囊血管成形术", "date": "2024-03-15"},
    {"code": "88.56", "name": "冠状动脉造影术", "date": "2024-03-14"}
  ],
  "patient_info": {
    "age": 65,
    "gender": "M",
    "length_of_stay": 7,
    "weight": 72.5
  },
  "options": {
    "version": "CN-DRG 2.0",
    "include_suggestions": true,
    "include_similar_groups": true
  }
}
```

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "prediction_id": "DRG20240315001",
    "patient_id": "P001",
    "visit_id": "V20240315001",
    "drg_group": {
      "code": "FM19",
      "name": "经皮心血管操作，伴CC",
      "mdc": "MDC05 循环系统疾病及功能障碍",
      "adrg": "FM1 经皮心血管操作",
      "weight": 1.456,
      "adr": 5,
      "relative_weight": 1.456
    },
    "grouping_details": {
      "primary_diagnosis": {"code": "I10", "name": "原发性高血压"},
      "main_procedure": {"code": "00.66", "name": "经皮冠状动脉球囊血管成形术"},
      "cc_list": [
        {"code": "E11.9", "name": "2型糖尿病", "type": "CC"},
        {"code": "I25.1", "name": "动脉粥样硬化性心脏病", "type": "CC"}
      ],
      "grouping_path": "MDC05 → FM1 → FM19"
    },
    "cost_analysis": {
      "currency": "CNY",
      "estimated_cost": 25000,
      "standard_cost": 28000,
      "reimbursement": 32000,
      "profit_margin": 7000,
      "profit_rate": 28.0,
      "risk_level": "low",
      "breakdown": {
        "medical_fee": 15000,
        "nursing_fee": 3000,
        "examination_fee": 5000,
        "medication_fee": 4000,
        "material_fee": 3000
      }
    },
    "suggestions": [
      {
        "type": "optimization",
        "priority": "high",
        "message": "建议完善术后心电图检查记录",
        "impact": "可提高分组准确性"
      },
      {
        "type": "warning",
        "priority": "medium",
        "message": "当前住院天数(7天)略低于ADR(5天)",
        "impact": "建议评估是否需要延长住院时间"
      },
      {
        "type": "info",
        "priority": "low",
        "message": "可考虑增加术后护理天数以优化费用结构",
        "impact": "优化费用分布"
      }
    ],
    "similar_groups": [
      {
        "code": "FM15",
        "name": "经皮心血管操作，不伴CC",
        "weight": 1.234,
        "difference": "无并发症/合并症"
      },
      {
        "code": "FM25",
        "name": "经皮心血管支架植入，伴CC",
        "weight": 2.156,
        "difference": "含支架植入"
      }
    ],
    "predicted_at": "2024-03-15T10:32:00Z",
    "processing_time": "1.5s",
    "confidence": 0.95
  }
}
```

---

### 2. 获取预测结果详情

**GET** `/api/v1/drg/{prediction_id}`

根据预测ID获取DRG分组预测详情。

#### 路径参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| prediction_id | string | 是 | 预测ID |

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "prediction_id": "DRG20240315001",
    "drg_group": {...},
    "cost_analysis": {...},
    "predicted_at": "2024-03-15T10:32:00Z"
  }
}
```

---

### 3. 费用分析

**POST** `/api/v1/drg/analyze`

对住院费用进行详细分析，提供成本优化建议。

#### 请求参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| prediction_id | string | 是 | DRG预测ID |
| actual_costs | object | 否 | 实际费用明细 |
| compare_standard | boolean | 否 | 是否与标准费用对比 |

#### 请求示例

```json
{
  "prediction_id": "DRG20240315001",
  "actual_costs": {
    "medical_fee": 14500,
    "nursing_fee": 2800,
    "examination_fee": 5500,
    "medication_fee": 4200,
    "material_fee": 3500
  },
  "compare_standard": true
}
```

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "analysis_id": "ANA20240315001",
    "prediction_id": "DRG20240315001",
    "summary": {
      "total_actual": 30500,
      "total_standard": 28000,
      "variance": 2500,
      "variance_rate": 8.9
    },
    "cost_comparison": {
      "medical_fee": {
        "actual": 14500,
        "standard": 15000,
        "variance": -500,
        "status": "under"
      },
      "nursing_fee": {
        "actual": 2800,
        "standard": 3000,
        "variance": -200,
        "status": "under"
      },
      "examination_fee": {
        "actual": 5500,
        "standard": 5000,
        "variance": 500,
        "status": "over"
      },
      "medication_fee": {
        "actual": 4200,
        "standard": 4000,
        "variance": 200,
        "status": "over"
      },
      "material_fee": {
        "actual": 3500,
        "standard": 3000,
        "variance": 500,
        "status": "over"
      }
    },
    "optimization_suggestions": [
      {
        "category": "examination_fee",
        "suggestion": "检查费用超出标准500元，建议评估检查项目的必要性",
        "potential_saving": 500
      },
      {
        "category": "material_fee",
        "suggestion": "材料费用超出标准500元，可考虑使用集采耗材",
        "potential_saving": 500
      }
    ],
    "reimbursement_analysis": {
      "expected_reimbursement": 32000,
      "actual_profit": 1500,
      "profit_rate": 4.9
    },
    "analyzed_at": "2024-03-15T10:33:00Z"
  }
}
```

---

### 4. 获取DRG分组列表

**GET** `/api/v1/drg/groups`

获取DRG分组列表，支持搜索和筛选。

#### 查询参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| mdc | string | 否 | MDC代码筛选 |
| adrg | string | 否 | ADRG代码筛选 |
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
        "code": "FM19",
        "name": "经皮心血管操作，伴CC",
        "mdc": "MDC05",
        "adrg": "FM1",
        "weight": 1.456,
        "adr": 5,
        "description": "经皮心血管介入治疗，伴有并发症或合并症"
      },
      {
        "code": "FM15",
        "name": "经皮心血管操作，不伴CC",
        "mdc": "MDC05",
        "adrg": "FM1",
        "weight": 1.234,
        "adr": 5,
        "description": "经皮心血管介入治疗，不伴并发症或合并症"
      }
    ],
    "pagination": {
      "page": 1,
      "page_size": 20,
      "total": 628,
      "total_pages": 32
    }
  }
}
```

---

### 5. 获取分组规则

**GET** `/api/v1/drg/rules`

获取DRG分组规则说明。

#### 查询参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| group_code | string | 否 | DRG组代码 |

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "rules": [
      {
        "rule_id": "DRG001",
        "name": "primary_diagnosis_priority",
        "display_name": "主诊断优先规则",
        "description": "主诊断决定DRG分组的MDC归属",
        "priority": 1
      },
      {
        "rule_id": "DRG002",
        "name": "procedure_combination",
        "display_name": "手术组合规则",
        "description": "多手术组合影响ADRG分组",
        "priority": 2
      },
      {
        "rule_id": "DRG003",
        "name": "cc_adjustment",
        "display_name": "并发症调整规则",
        "description": "并发症/合并症影响最终DRG组",
        "priority": 3
      }
    ]
  }
}
```

---

## DRG分组说明

### MDC（主要诊断大类）

| MDC代码 | 名称 |
|---------|------|
| MDC01 | 神经系统疾病及功能障碍 |
| MDC02 | 眼部疾病及功能障碍 |
| MDC03 | 耳鼻咽喉疾病及功能障碍 |
| MDC04 | 呼吸系统疾病及功能障碍 |
| MDC05 | 循环系统疾病及功能障碍 |
| MDC06 | 消化系统疾病及功能障碍 |
| ... | ... |

### CC/MCC 说明

| 类型 | 说明 |
|------|------|
| MCC | 主要并发症或合并症，显著影响分组权重 |
| CC | 并发症或合并症，适度影响分组权重 |
| Non-CC | 非并发症，不影响分组权重 |

---

## 错误码

| 错误码 | 说明 |
|--------|------|
| 40001 | 患者ID不存在 |
| 40002 | 诊断编码无效 |
| 40003 | 手术编码无效 |
| 40004 | DRG版本不支持 |
| 40401 | 预测结果不存在 |
| 50001 | 分组引擎执行失败 |

---

<div align="center">

**[返回API目录](./README.md)**

</div>
