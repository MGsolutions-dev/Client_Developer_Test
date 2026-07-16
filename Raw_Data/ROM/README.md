# ROM Raw Data — Anterior_Skeleton.json 데이터 명세

이 문서는 `Normal_Data/`, `Abnormal_Data/` 폴더 안의 **Anterior_Skeleton.json** 파일에 기록된
관절(Joint) 데이터의 구조와, 숫자로 표기된 `type` 값이 어떤 관절(JointType)을 의미하는지 설명합니다.

`type` 값의 정의 출처는 Moti-Physio 2 프로젝트의
`Assets/SensorManager/Scripts/Manager/SkeletonFrame.cs` 내부 `SensorSkeleton.Joint.JointType` enum 입니다.

---

## 1. 파일 개요

| 파일 | 설명 |
|---|---|
| `Anterior_Skeleton.json` | 전면(Anterior) 촬영 시 프레임별로 기록된 스켈레톤(관절 좌표) 데이터 배열 |
| `Color_CaliInfo.json` | 컬러 카메라 캘리브레이션 정보 (focalLength, cameraAngle, frameSize 등) |
| `DeepDepth_CaliInfo.json` | 깊이(Depth) 카메라 캘리브레이션 정보 |

| 데이터셋 | 프레임 수 |
|---|---|
| `Normal_Data/Anterior_Skeleton.json` | 779 프레임 |
| `Abnormal_Data/Anterior_Skeleton.json` | 739 프레임 |

## 2. JSON 구조

`Anterior_Skeleton.json`은 **프레임 단위 스켈레톤 객체의 배열**입니다.
각 원소는 `SensorSkeleton` 클래스가 직렬화된 형태입니다.

```json
[
  {
    "ID": 2,                        // 해당 프레임에서 인식된 Skeleton의 ID
    "NowDataTimestamp": 1784185538, // 데이터 기록 시각 (Unix Time, 초 단위)
    "Joints": [                     // 관절 배열
      {
        "type": 2,                  // 관절 타입 (아래 JointType 표 참조)
        "confidence": 0.75,         // 관절 인식 신뢰도 (0.0 ~ 1.0)
        "REAL": {                   // 실제 3D 좌표 (카메라 기준 공간 좌표, mm 단위)
          "x": -30.31, "y": 594.30, "z": 2171.61
        },
        "PROJ": {                   // 화면 투영 좌표 (x, y는 0~1 정규화 값, z는 깊이)
          "x": 0.483, "y": 0.747, "z": 2171.61
        }
      }
    ]
  }
]
```

### 필드 설명

| 필드 | 타입 | 설명 |
|---|---|---|
| `ID` | int | 센서가 부여한 스켈레톤(사람) 식별자 |
| `NowDataTimestamp` | long | 프레임 기록 시각. `ToUnixTimeSeconds()` 기반 Unix Timestamp(초) |
| `Joints[].type` | int | 관절 종류. `SensorSkeleton.Joint.JointType` enum 값 (아래 표) |
| `Joints[].confidence` | float | 관절 인식 신뢰도. `0.0` = 인식 실패/미지원, 값이 클수록 신뢰도 높음 |
| `Joints[].REAL` | Vector3 | 센서 기준 실제 3D 공간 좌표 (mm). z는 카메라로부터의 거리 |
| `Joints[].PROJ` | Vector3 | 프레임에 투영된 2D 좌표. x, y는 0~1로 정규화, z는 REAL의 깊이 값과 동일 |

## 3. JointType 매핑 표 (`type` 숫자 → 관절)

`SkeletonFrame.cs`의 `SensorSkeleton.Joint.JointType` enum 전체 정의입니다.
접두사 `L_` = 왼쪽(Left), `R_` = 오른쪽(Right)이며, **환자(피촬영자) 본인 기준의 좌/우**입니다.

### 특수 값

| type | Enum 이름 | 한글 명칭 | 설명 |
|---:|---|---|---|
| 0 | `None` | 없음 | 관절 아님(더미/자리 표시용). confidence가 0.0으로 기록됨 |
| 1 | `NotSupport` | 미지원 | 현재 센서에서 지원하지 않는 관절 슬롯 |

### 머리 · 몸통 (체간)

| type | Enum 이름 | 한글 명칭 | 설명 |
|---:|---|---|---|
| 2 | `Head` | 머리 | 두부 중심점 |
| 3 | `Neck` | 목 | 경부 |
| 4 | `Torso` | 몸통 | 흉추 부근 몸통 중심. 코드에서 기준 관절(default target)로 사용됨 |
| 5 | `Waist` | 허리 | 요추/골반 상단 부근 몸통 하단 중심 |

### 어깨 라인 (쇄골 · 어깨)

| type | Enum 이름 | 한글 명칭 | 설명 |
|---:|---|---|---|
| 6 | `L_Collar` | 왼쪽 칼라(쇄골 안쪽) | 목 아래 왼쪽 쇄골 시작점 |
| 7 | `R_Collar` | 오른쪽 칼라(쇄골 안쪽) | 목 아래 오른쪽 쇄골 시작점 |
| 8 | `L_Clavicle` | 왼쪽 쇄골 | 왼쪽 쇄골 (본 데이터에는 미출현) |
| 9 | `R_Clavicle` | 오른쪽 쇄골 | 오른쪽 쇄골 (본 데이터에는 미출현) |
| 10 | `L_Shoulder` | 왼쪽 어깨 | 왼쪽 견관절 |
| 11 | `R_Shoulder` | 오른쪽 어깨 | 오른쪽 견관절 |

### 팔 (상지)

| type | Enum 이름 | 한글 명칭 | 설명 |
|---:|---|---|---|
| 12 | `L_Elbow` | 왼쪽 팔꿈치 | 왼쪽 주관절 |
| 13 | `R_Elbow` | 오른쪽 팔꿈치 | 오른쪽 주관절 |
| 14 | `L_Wrist` | 왼쪽 손목 | 왼쪽 수관절 |
| 15 | `R_Wrist` | 오른쪽 손목 | 오른쪽 수관절 |
| 16 | `L_Hand` | 왼쪽 손 | 왼쪽 손 중심점 |
| 17 | `R_Hand` | 오른쪽 손 | 오른쪽 손 중심점 |

### 다리 (하지)

| type | Enum 이름 | 한글 명칭 | 설명 |
|---:|---|---|---|
| 18 | `L_Hip` | 왼쪽 엉덩이(고관절) | 왼쪽 고관절 |
| 19 | `R_Hip` | 오른쪽 엉덩이(고관절) | 오른쪽 고관절 |
| 20 | `L_Knee` | 왼쪽 무릎 | 왼쪽 슬관절 |
| 21 | `R_Knee` | 오른쪽 무릎 | 오른쪽 슬관절 |
| 22 | `L_Ankle` | 왼쪽 발목 | 왼쪽 족관절 |
| 23 | `R_Ankle` | 오른쪽 발목 | 오른쪽 족관절 |
| 24 | `L_Foot` | 왼쪽 발 | 왼쪽 발끝/발 중심점 |
| 25 | `R_Foot` | 오른쪽 발 | 오른쪽 발끝/발 중심점 |

## 4. 본 데이터에서의 사용 현황

Normal / Abnormal 두 데이터셋 모두 프레임마다 아래 24개 type이 기록되어 있습니다.

- **출현하는 type**: `0, 1, 2, 3, 4, 5, 6, 7, 10~25`
- **출현하지 않는 type**: `8 (L_Clavicle)`, `9 (R_Clavicle)` — 사용된 센서가 쇄골 관절을 별도로 제공하지 않음
- `type 0 (None)`, `type 1 (NotSupport)`은 실제 관절이 아니라 자리 표시용 슬롯이며,
  `confidence`가 `0.0`이고 좌표가 무의미하므로 **분석 시 제외**해야 합니다.

## 5. 스켈레톤 연결 구조 (참고)

관절을 선으로 이었을 때의 인체 구조는 다음과 같습니다.

```
                 Head(2)
                   │
                 Neck(3)
        ┌──────────┼──────────┐
  L_Collar(6)      │      R_Collar(7)
        │          │          │
 L_Shoulder(10)    │     R_Shoulder(11)
        │       Torso(4)      │
  L_Elbow(12)      │     R_Elbow(13)
        │       Waist(5)      │
  L_Wrist(14)      │     R_Wrist(15)
        │     ┌────┴────┐     │
   L_Hand(16) │         │ R_Hand(17)
         L_Hip(18)  R_Hip(19)
              │         │
        L_Knee(20)  R_Knee(21)
              │         │
        L_Ankle(22) R_Ankle(23)
              │         │
        L_Foot(24)  R_Foot(25)
```

---
