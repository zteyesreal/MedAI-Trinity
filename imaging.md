# 影像分析接口文档

<div align="center">

[![Module](https://img.shields.io/badge/Module-imaging-blue.svg)](https://github.com)
[![Version](https://img.shields.io/badge/Version-v1.0-green.svg)](https://github.com)

</div>

---

## 模块概述

影像分析模块提供PACS影像智能分析功能，支持CT、MRI、X光等多种影像类型的自动识别和分析。

**端点前缀：** `/api/v1/imaging`

---

## 接口列表

| 方法 | 端点 | 说明 |
|------|------|------|
| POST | `/analyze` | 影像智能分析 |
| GET | `/{analysis_id}` | 获取分析结果详情 |
| POST | `/compare` | 影像对比分析 |
| GET | `/studies` | 获取影像检查列表 |
| GET | `/models` | 获取可用分析模型 |

---

## 接口详情

### 1. 影像智能分析

**POST** `/api/v1/imaging/analyze`

对PACS影像进行智能分析并生成报告。

#### 请求参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| patient_id | string | 是 | 患者ID |
| study_id | string | 是 | 检查ID |
| dicom_files | array | 是 | DICOM文件列表 |
| study_type | string | 是 | 检查类型 |
| options | object | 否 | 分析选项 |

#### study_type 可选值

| 值 | 说明 |
|------|------|
| CT_HEAD | 头颅CT |
| CT_CHEST | 胸部CT |
| CT_ABDOMEN | 腹部CT |
| CT_SPINE | 脊柱CT |
| MRI_BRAIN | 脑部MRI |
| MRI_SPINE | 脊柱MRI |
| XRAY_CHEST | 胸部X光 |
| XRAY_BONE | 骨骼X光 |

#### options 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| detect_nodules | boolean | 否 | 是否检测结节，默认true |
| detect_fractures | boolean | 否 | 是否检测骨折 |
| detect_lesions | boolean | 否 | 是否检测病变 |
| compare_previous | boolean | 否 | 是否与历史影像对比 |
| generate_report | boolean | 否 | 是否生成报告，默认true |
| confidence_threshold | float | 否 | 置信度阈值（0-1），默认0.7 |

#### 请求示例

```json
{
  "patient_id": "P001",
  "study_id": "ST20240315001",
  "dicom_files": [
    "CT_CHEST_001.dcm",
    "CT_CHEST_002.dcm",
    "CT_CHEST_003.dcm"
  ],
  "study_type": "CT_CHEST",
  "options": {
    "detect_nodules": true,
    "detect_lesions": true,
    "compare_previous": true,
    "generate_report": true,
    "confidence_threshold": 0.75
  }
}
```

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "analysis_id": "IMG20240315001",
    "patient_id": "P001",
    "study_id": "ST20240315001",
    "study_type": "CT_CHEST",
    "findings": [
      {
        "id": "F001",
        "location": "右肺上叶尖段",
        "type": "结节",
        "subtype": "磨玻璃结节",
        "size": {
          "length": 8.5,
          "width": 6.2,
          "unit": "mm"
        },
        "volume": 215.6,
        "density": "磨玻璃影",
        "margin": "光滑",
        "shape": "类圆形",
        "calcification": false,
        "enhancement": "无强化",
        "risk_level": "moderate",
        "risk_score": 0.45,
        "classification": "Lung-RADS 3",
        "recommendation": "建议3个月后复查胸部CT",
        "confidence": 0.92,
        "bbox": {
          "x": 156,
          "y": 89,
          "width": 32,
          "height": 28
        }
      },
      {
        "id": "F002",
        "location": "左肺下叶背段",
        "type": "结节",
        "subtype": "实性结节",
        "size": {
          "length": 4.2,
          "width": 3.8,
          "unit": "mm"
        },
        "volume": 35.2,
        "density": "实性",
        "margin": "光滑",
        "shape": "类圆形",
        "calcification": true,
        "risk_level": "low",
        "risk_score": 0.15,
        "classification": "Lung-RADS 2",
        "recommendation": "年度随访即可",
        "confidence": 0.88
      },
      {
        "id": "F003",
        "location": "气管隆突下",
        "type": "淋巴结",
        "size": {
          "length": 12.3,
          "width": 8.5,
          "unit": "mm"
        },
        "density": "软组织密度",
        "risk_level": "low",
        "recommendation": "大小正常范围，建议随访观察",
        "confidence": 0.85
      }
    ],
    "comparison": {
      "has_previous": true,
      "previous_study_id": "ST20231215001",
      "previous_date": "2023-12-15",
      "changes": [
        {
          "finding_id": "F001",
          "change_type": "new",
          "description": "新发现的磨玻璃结节",
          "significance": "需要关注"
        },
        {
          "finding_id": "F002",
          "change_type": "stable",
          "description": "与前片对比无明显变化",
          "significance": "稳定"
        }
      ]
    },
    "report": {
      "examination": "胸部CT平扫",
      "technique": "层厚5mm，螺距1.0",
      "findings_text": "双肺纹理清晰，右肺上叶尖段见一磨玻璃结节，大小约8.5mm×6.2mm，边缘光滑。左肺下叶背段见一实性小结节，大小约4.2mm×3.8mm，内见钙化。气管及主要支气管通畅。纵隔内气管隆突下见小淋巴结，短径约8.5mm。双侧胸腔未见积液。心脏大小形态正常。",
      "impression": "1. 右肺上叶磨玻璃结节，Lung-RADS 3类，建议3个月后复查CT\n2. 左肺下叶钙化小结节，Lung-RADS 2类，年度随访\n3. 纵隔小淋巴结，建议随访观察",
      "recommendations": [
        "3个月后复查胸部CT",
        "必要时行PET-CT检查评估磨玻璃结节性质",
        "年度随访左肺下叶结节"
      ]
    },
    "quality_metrics": {
      "image_quality": "good",
      "motion_artifact": "none",
      "noise_level": "low",
      "contrast": "adequate"
    },
    "analyzed_at": "2024-03-15T10:35:00Z",
    "processing_time": "12.5s",
    "model_version": "imaging-v2.1"
  }
}
```

---

### 2. 获取分析结果详情

**GET** `/api/v1/imaging/{analysis_id}`

根据分析ID获取影像分析结果详情。

#### 路径参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| analysis_id | string | 是 | 分析ID |

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "analysis_id": "IMG20240315001",
    "findings": [...],
    "report": {...},
    "analyzed_at": "2024-03-15T10:35:00Z"
  }
}
```

---

### 3. 影像对比分析

**POST** `/api/v1/imaging/compare`

对比两次影像检查的变化。

#### 请求参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| current_study_id | string | 是 | 当前检查ID |
| previous_study_id | string | 是 | 历史检查ID |
| focus_areas | array | 否 | 关注区域列表 |

#### 请求示例

```json
{
  "current_study_id": "ST20240315001",
  "previous_study_id": "ST20231215001",
  "focus_areas": ["lung_nodules", "lymph_nodes"]
}
```

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "comparison_id": "CMP20240315001",
    "current_study": {
      "study_id": "ST20240315001",
      "study_date": "2024-03-15"
    },
    "previous_study": {
      "study_id": "ST20231215001",
      "study_date": "2023-12-15"
    },
    "interval_days": 90,
    "changes": [
      {
        "type": "new_finding",
        "location": "右肺上叶尖段",
        "description": "新发现磨玻璃结节，大小8.5mm×6.2mm",
        "clinical_significance": "需要关注，建议短期随访"
      },
      {
        "type": "stable",
        "location": "左肺下叶背段",
        "description": "实性结节大小无明显变化",
        "size_change": "0.2mm",
        "clinical_significance": "稳定，继续随访"
      }
    ],
    "summary": {
      "new_findings": 1,
      "resolved_findings": 0,
      "changed_findings": 0,
      "stable_findings": 1
    }
  }
}
```

---

### 4. 获取影像检查列表

**GET** `/api/v1/imaging/studies`

获取患者的影像检查列表。

#### 查询参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| patient_id | string | 是 | 患者ID |
| study_type | string | 否 | 检查类型筛选 |
| start_date | string | 否 | 开始日期 |
| end_date | string | 否 | 结束日期 |
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
        "study_id": "ST20240315001",
        "study_type": "CT_CHEST",
        "study_date": "2024-03-15",
        "status": "analyzed",
        "findings_count": 3
      },
      {
        "study_id": "ST20231215001",
        "study_type": "CT_CHEST",
        "study_date": "2023-12-15",
        "status": "analyzed",
        "findings_count": 1
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

### 5. 获取可用分析模型

**GET** `/api/v1/imaging/models`

获取系统支持的分析模型列表。

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "models": [
      {
        "model_id": "lung_nodule_detection",
        "name": "肺结节检测模型",
        "version": "v2.1",
        "study_types": ["CT_CHEST"],
        "accuracy": 0.95,
        "sensitivity": 0.92,
        "specificity": 0.97
      },
      {
        "model_id": "fracture_detection",
        "name": "骨折检测模型",
        "version": "v1.5",
        "study_types": ["XRAY_BONE", "CT_SPINE"],
        "accuracy": 0.93,
        "sensitivity": 0.90,
        "specificity": 0.95
      },
      {
        "model_id": "brain_lesion_detection",
        "name": "脑部病变检测模型",
        "version": "v1.8",
        "study_types": ["CT_HEAD", "MRI_BRAIN"],
        "accuracy": 0.91,
        "sensitivity": 0.88,
        "specificity": 0.94
      }
    ]
  }
}
```

---

## Lung-RADS 分类说明

| 分类 | 说明 | 处理建议 |
|------|------|----------|
| 1类 | 阴性 | 年度筛查 |
| 2类 | 良性发现 | 年度筛查 |
| 3类 | 良性可能 | 6个月后复查 |
| 4A类 | 低度恶性可能 | 3个月后复查或穿刺 |
| 4B类 | 中度恶性可能 | 进一步检查或穿刺 |
| 4C类 | 高度恶性可能 | 积极干预 |
| 5类 | 高度提示恶性 | 手术或其他治疗 |

---

## 错误码

| 错误码 | 说明 |
|--------|------|
| 40001 | 患者ID不存在 |
| 40002 | 检查ID不存在 |
| 40003 | DICOM文件格式错误 |
| 40004 | 检查类型不支持 |
| 40401 | 分析结果不存在 |
| 50001 | 影像分析引擎执行失败 |

---

<div align="center">

**[返回API目录](./README.md)**

</div>
