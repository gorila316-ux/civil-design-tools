# Civil 3D 2026 API 변경사항

> 2025 대비 2026 버전의 API 변경 내역

## 개요

- **출시일**: 2025년 3월
- **ObjectARX 버전**: 2026
- **AeccDbMgd 버전**: 15.0.0.0

## 신규 기능

### 1. TBD - 문서화 예정

*실제 API 변경사항은 ObjectARX SDK Release Notes 참조*

## Deprecated (지원 중단 예정)

| API | 대체 API | 비고 |
|-----|----------|------|
| TBD | TBD | - |

## Breaking Changes

*2025 → 2026 마이그레이션 시 확인 필요*

| 변경 항목 | 영향 | 해결 방법 |
|-----------|------|-----------|
| 어셈블리 버전 변경 | 재컴파일 필요 | 2026 SDK로 빌드 |
| TBD | TBD | TBD |

## 마이그레이션 가이드

### 2025 → 2026

1. **참조 업데이트**: 모든 Autodesk DLL 참조를 2026 버전으로 변경
2. **재컴파일**: x64, .NET 4.8 대상으로 빌드
3. **테스트**: 주요 기능 동작 확인

### 호환성 체크리스트

- [ ] 모든 참조 DLL 버전 확인
- [ ] Deprecated API 사용 여부 확인
- [ ] Breaking Changes 영향 분석
- [ ] 테스트 케이스 실행

## 외부 참조

- [AutoCAD 2026 What's New](https://help.autodesk.com/view/ACD/2026/KOR/)
- [Civil 3D 2026 What's New](https://help.autodesk.com/view/CIV3D/2026/KOR/)
- [ObjectARX 2026 Release Notes](https://help.autodesk.com/view/OARX/2026/KOR/)

---

*이 문서는 실제 API 변경사항 확인 후 업데이트 예정*
