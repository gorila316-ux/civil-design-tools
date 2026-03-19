# Civil Design Tools

> 토목 설계 업무 자동화 도구 모음

## 개요

Civil3D 기반 토목 설계 업무의 생산성을 높이기 위한 WPF 도구 모음입니다.
C# 학습과 실무를 연계하여 단계별로 개발합니다.

## 개발 현황

| Phase | 도구명 | 상태 | 관련 학습 |
|-------|--------|------|-----------|
| 1 | 도면 파일 현황 조회 | 예정 | Ch 01 |
| 2 | 업무 체크리스트 | 예정 | Ch 02~03 |
| 3 | 토목 구조물 수량 계산기 | 예정 | Ch 04~05 |
| 4 | 구조물 유형별 설계 계산기 | 예정 | Ch 06~07 |
| 5 | 설계 데이터 조회/분석 도구 | 예정 | Ch 08~09 |
| 6 | 수량 산출서 Excel 자동 출력 | 예정 | Ch 10 |
| 7 | 암거 자동배치 프로토타입 | 예정 | Ch 12 (최종) |

## 저장소 구조

```
civil-design-tools/
├── README.md              ← 도구 소개 + 개발 현황
├── docs/
│   └── progress.md        ← 주차별 성과 기록
├── Phase1-FileManager/    ← 도면 파일 관리
├── Phase2-TaskManager/    ← 업무 체크리스트
├── Phase3-Calculator/     ← 수량 계산기
├── Phase4-StructCalc/     ← 구조물 계산기
├── Phase5-DataAnalysis/   ← 데이터 분석
├── Phase6-ExcelExport/    ← Excel 출력
└── Phase7-CulvertTool/    ← 암거 자동배치
```

## 기술 스택

- **Language**: C# (.NET)
- **UI Framework**: WPF (Windows Presentation Foundation)
- **Libraries**: EPPlus (Excel), Newtonsoft.Json
- **Target**: AutoCAD/Civil3D API 연동 (Phase 7)

## 개발 원칙

1. **학습 완료 후 적용** - 개념 이해 없이 바이브코딩 금지
2. **핵심 주석 직접 작성** - 이해한 것만 주석으로 기록
3. **기능 단위 완성** - 주차별 성과 보고 가능한 결과물

## 연관 저장소

- [Csharp-Learning](https://github.com/gorila316-ux/Csharp-Learning) - C# 학습 저장소

---

*개발 시작: 2026-03*
