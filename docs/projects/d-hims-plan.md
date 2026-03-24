# D-HIMS: Digital Twin Home Inventory Management System

> **프로젝트명**: D-HIMS (Digital Twin Home Inventory Management System)
> **목적**: BIM IFC 모델 기반 가정 내 수납공간/냉장고 재고 디지털트윈 관리
> **성격**: 응용 파일럿 프로젝트 (C# 학습 연계 + 웹 기술 확장)
> **예정 저장소**: `home-digital-twin` (별도 생성 예정)
> **작성일**: 2025-03-24

---

## 프로젝트 비전

BIM(IFC)의 3D 공간 데이터와 실시간 재고(Inventory) 데이터를 결합하여,
가정 내 모든 자산과 식자재를 시각적으로 추적하고 관리하는
가정용 디지털 트윈(Digital Twin) 환경 구축.

### 확장 가능성
```
현재: 가정용 수납공간 재고 관리
  ↓ 동일한 아키텍처로 치환
미래 1: 암거 유지관리 디지털트윈
        (암거 GlobalId → 점검이력/상태 관리)
미래 2: 교량 부재별 이력관리 시스템
        (교량 부재 GlobalId → 균열/보수 이력)
미래 3: Civil3D 시설물 유지관리 플랫폼
        (civil-design-tools와 통합)
```

---

## 시스템 아키텍처

```
┌─────────────────────────────────────────────────────┐
│                    Frontend                          │
│  React + ThatOpen Engine (IFC 뷰어) + Tailwind CSS  │
│  - IFC 3D 뷰어 (web-ifc / Three.js)                │
│  - 검색 → ExpressID → 3D 하이라이트 연동            │
│  - 재고 목록/검색/필터 UI                            │
└──────────────────────┬──────────────────────────────┘
                       │ REST API
┌──────────────────────▼──────────────────────────────┐
│                    Backend                           │
│  ASP.NET Core (.NET 8) Web API (C#)                 │
│  - IFC 파일 파싱 (xBIM Toolkit)                     │
│  - GlobalId 기반 DB 매핑                             │
│  - 재고 CRUD API                                     │
│  - 알림 스케줄러 (메일 발송)                         │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│                   Database                           │
│  Google Cloud Firestore                              │
│  - IFC 속성 데이터 (GlobalId 키값)                  │
│  - 재고 정보 실시간 동기화                           │
│  - onSnapshot 리스너 → 전 단말 즉시 반영            │
└─────────────────────────────────────────────────────┘
```

---

## 기술 스택

| 영역 | 기술 | 비고 |
|------|------|------|
| **Frontend** | React, ThatOpen Engine, Three.js, Tailwind CSS | IFC 뷰어 |
| **Backend** | ASP.NET Core (.NET 8), xBIM Toolkit | C# 학습과 일치 |
| **Database** | Google Cloud Firestore | 실시간 동기화 |
| **Notification** | SendGrid API | 메일 발송 |

---

## Core 기능

### 1. IFC 모델 뷰어
- 집 평면도 or 수납공간 3D 모델 표시
- 공간 클릭 시 해당 재고 조회
- 검색 시 해당 3D 객체 하이라이트

### 2. 재고 DB 관리
- IFC GlobalId를 키값으로 DB 매핑
- IfcPropertySet 스캔 → 속성 추출
- 품목명, 수량, 유효기간, 위치 저장

### 3. 검색/조회/필터링
- 품목명 검색 → ExpressID 매칭 → 3D 하이라이트
- 유효기간 필터 (임박/만료)
- 위치별 필터 (냉장고/수납장/팬트리)

### 4. 실시간 동기화
- 3D 객체 클릭 → 팝업창에서 수량 수정
- Firestore onSnapshot으로 전 단말 즉시 동기화

### 5. 알림 시스템
- ExpiryDate <= Today+3 → 메일 알림
- CurrentStock < MinimumStock → 재고 부족 알림

---

## C# 학습 연계 포인트

```
Ch 02~03  →  InventoryItem 클래스 설계
Ch 04~05  →  속성(Property)으로 유효기간/수량 관리
Ch 06     →  StorageSpace 상속 구조
             (냉장고, 수납장, 팬트리 클래스 계층)
Ch 07     →  IStorable 인터페이스 정의
             INotifyPropertyChanged → 실시간 UI 갱신
Ch 08     →  List/Dictionary로 재고 컬렉션 관리
Ch 09     →  LINQ로 유효기간 임박 항목 쿼리
             → ExpiryDate <= Today+3
Ch 10     →  IFC 파일 읽기/JSON 직렬화
Ch 12     →  API 에러 처리 (try/catch/finally)
```

### C# 설계 패턴 예시
```csharp
// Ch 07 인터페이스 학습과 직접 연계
public interface IStorable
{
    string GlobalId { get; }
    string Name { get; set; }
    List<InventoryItem> Items { get; set; }
    void SyncToFirestore();
}

// Ch 06 상속 학습과 연계
public abstract class StorageSpace : IStorable { ... }
public class Refrigerator : StorageSpace { ... }
public class Cabinet : StorageSpace { ... }
public class Pantry : StorageSpace { ... }

// Ch 09 LINQ 학습과 연계
var expiringSoon = items
    .Where(i => i.ExpiryDate <= DateTime.Today.AddDays(3))
    .OrderBy(i => i.ExpiryDate)
    .ToList();
```

---

## 개발 단계 계획

### Phase 0 — IFC 모델 준비 (즉시 시작 가능)
```
□ 집 평면도 IFC 모델 작성 (FreeCAD 또는 Blender + BlenderBIM)
□ IFC 속성 설계 (GlobalId 기반)
□ home-digital-twin GitHub 저장소 생성
```

### Phase 1 — IFC 뷰어 + 속성 추출 (C# Ch 01~03 완료 후)
```
□ React 프로젝트 생성
□ ThatOpen Components 설치
□ IFC 파일 로드 + 3D 표시
□ 공간 클릭 → GlobalId → 속성 조회
```

### Phase 2 — DB 연동 + 조회 (C# Ch 04~06 완료 후)
```
□ ASP.NET Core Web API 프로젝트
□ xBIM으로 IFC 파싱 + GlobalId 추출
□ Firestore 컬렉션 설계
□ 재고 CRUD API 구현
□ IStorable 인터페이스 + 상속 구조 설계
```

### Phase 3 — 양방향 인터랙션 + 알림 (C# Ch 07~09 완료 후)
```
□ 검색 → ExpressID → 3D 하이라이트 연동
□ 3D 클릭 → 팝업 → 수량 수정
□ onSnapshot으로 실시간 동기화
□ LINQ로 유효기간 임박 항목 쿼리
□ SendGrid 메일 발송 연동
```

### Phase 4 — 알림 고도화 (C# Ch 10~12 완료 후)
```
□ 유효기간 만료 스케줄러 (D-7, D-3, D-1)
□ 재고 부족 자동 알림
□ 에러 처리 완비 (try/catch)
```

### Phase 5 — 클라우드 배포 (전체 완료 후)
```
□ Firebase Hosting 배포
□ 모바일 반응형 UI 최적화
□ 사용자 인증 추가
```

---

## 참고 GitHub 오픈소스

### IFC 뷰어 핵심
| 프로젝트 | 설명 | 링크 |
|----------|------|------|
| ThatOpen Engine | IFC 파싱 (JavaScript) | github.com/ThatOpen/engine_web-ifc |
| ThatOpen Components | IFC 뷰어 UI 컴포넌트 | github.com/ThatOpen/engine_components |
| web-ifc-viewer | 완성된 IFC 뷰어 예제 | github.com/ThatOpen/web-ifc-viewer |

### IFC 파싱 (.NET/C#)
| 프로젝트 | 설명 | 링크 |
|----------|------|------|
| xBIM Toolkit | C#/.NET IFC 파싱 | github.com/xBimTeam/XbimEssentials |
| BIMserver | IFC 데이터베이스 서버 | github.com/opensourceBIM/BIMserver |

### 참고 예제
| 프로젝트 | 설명 | 링크 |
|----------|------|------|
| IFCStudio | ThatOpen + ASP.NET Core 조합 | github.com/Ibrahimgabrieel/IFCStudio |

---

## 예상 저장소 구조

```
home-digital-twin/
├── README.md
├── docs/
│   ├── plan.md             ← 이 문서 이동
│   └── ifc-schema.md       ← IFC 속성 설계
├── frontend/               ← React + ThatOpen
│   ├── src/
│   │   ├── components/
│   │   └── services/
│   └── package.json
├── backend/                ← ASP.NET Core (.NET 8)
│   ├── Controllers/
│   ├── Models/
│   └── Services/
├── models/
│   └── home-model.ifc
└── database/
    └── firestore-schema.md
```

---

## civil-design-tools와의 관계

```
civil-design-tools          home-digital-twin
─────────────────           ──────────────────
Civil 3D 플러그인           BIM/IFC 웹앱
.NET 8 + Civil API          React + ASP.NET Core
WPF UI                      웹 UI
        ↘                  ↙
         xBIM(IFC) 경험 공유
         IStorable 패턴 공유
         디지털트윈 아키텍처 응용
```

---

*관련: [CLAUDE.md](../../CLAUDE.md) | [csharp-master-plan.md](../../../Csharp-Learning/csharp-master-plan.md)*
