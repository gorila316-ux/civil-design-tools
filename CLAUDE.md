# CLAUDE.md - civil-design-tools

> Claude Code 세션 간 맥락 유지를 위한 프로젝트 컨텍스트

## 프로젝트 개요
- **목적**: Civil 3D 실무 자동화 도구 개발
- **대상 버전**: Civil 3D 2025, 2026
- **플랫폼**: .NET 8.0

## 저장소 구조
```
civil-design-tools/
├── CLAUDE.md                    ← 이 파일
├── docs/
│   ├── api-reference/           ← API 레퍼런스 문서 (22개)
│   │   ├── README.md            ← 메인 인덱스
│   │   ├── common/              ← 개발환경, 프로젝트 구조, 배포
│   │   ├── 2025/                ← 어셈블리 버전, CHANGELOG
│   │   ├── 2026/                ← 어셈블리 버전, CHANGELOG
│   │   ├── autocad-api/         ← Transaction, DB, Entity, Selection, Geometry
│   │   ├── civil3d-api/         ← Alignment, Profile, Surface, Corridor, Pipe
│   │   ├── code-samples/basic/  ← Hello Civil3D 예제
│   │   └── appendix/            ← DXF, 시스템변수, 외부링크
│   └── projects/
│       └── d-hims-plan.md       ← D-HIMS 프로젝트 계획 (별도 저장소 예정)
└── (Phase 폴더들 - 추후 생성)
```

## API 레퍼런스 활용
```
Claude Code가 docs/api-reference/ 참조
→ 정확한 Civil3D .NET 코드 생성
→ 환각(hallucination) 최소화

주요 문서:
- autocad-api/transaction.md     ← Transaction 패턴 (필수)
- civil3d-api/alignment.md       ← 선형 객체 접근
- 2025/assembly-versions.md      ← DLL 버전 (.NET 8.0)
```

---

## 🗺️ 파일럿 프로젝트 로드맵

### 단기 프로젝트 (C# 학습 연계)
> 목표: C# 교재 진도와 병행, WPF 기반 독립 실행 도구

| Phase | 프로젝트 | 학습 연계 | 예상 시점 |
|-------|----------|-----------|-----------|
| 1 | 도면 파일 현황 조회 | Ch 01 (WPF Grid, foreach) | Month 3 |
| 2 | 업무 체크리스트 | Ch 02-03 (클래스, 변수) | Month 3 |
| 3 | 토목 구조물 수량 계산기 | Ch 04-05 (타입, 캡슐화) | Month 4 |
| 4 | 구조물 유형별 설계 계산기 | Ch 06-07 (상속, 인터페이스) | Month 4 |
| 5 | 설계 데이터 조회/분석 | Ch 08-09 (컬렉션, LINQ) | Month 5 |
| 6 | 수량 산출서 Excel 출력 | Ch 10 (파일, JSON, EPPlus) | Month 5 |

### 중기 프로젝트 (Civil 3D API 연계)
> 목표: Civil 3D 플러그인 개발, AutoCAD/Civil 3D API 활용

| Phase | 프로젝트 | 기술 요소 | 예상 시점 |
|-------|----------|-----------|-----------|
| 7 | 암거 자동배치 프로토타입 ⭐ | Corridor, Alignment, Transaction | Month 6-7 |
| 8 | 선형/종단 데이터 추출기 | Alignment, Profile API | Month 7-8 |
| 9 | 지형 분석 도구 | TinSurface, 고도 쿼리 | Month 8 |
| 10 | 관망 자동 생성 | Pipe Network API | Month 9 |

### 장기 프로젝트 (확장)
> 목표: 디지털 트윈, BIM 연계

| 프로젝트 | 설명 | 비고 |
|----------|------|------|
| D-HIMS | BIM IFC 기반 재고 관리 디지털트윈 | 별도 저장소 예정 |
| 시설물 유지관리 | 암거/교량 점검이력 관리 | D-HIMS 패턴 응용 |

---

## 연관 저장소

### 학습 저장소
- **Csharp-Learning**: C# 학습 프로젝트
  - `csharp-master-plan.md`: Month 5-6에 api-reference 활용 체크리스트
  - `learning-notes.md`: 일별 학습 기록

### 예정 저장소
- **home-digital-twin**: D-HIMS 프로젝트 (BIM + IFC + Firestore)
  - 상세 계획: `docs/projects/d-hims-plan.md`

---

## 최근 작업 (2025-03-24)
- API 레퍼런스 문서 체계 구축 완료 (22개 문서)
- appendix 섹션 추가 (DXF, 시스템변수, 외부링크)
- 파일럿 로드맵 단기/중기/장기 재구성
- D-HIMS 프로젝트 계획 문서화

## 다음 세션 시작 시
```
"CLAUDE.md 읽고 현재 상태 파악해줘"
```

---
*최종 업데이트: 2025-03-24*
