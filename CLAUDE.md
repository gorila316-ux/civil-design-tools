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
│   └── api-reference/           ← API 레퍼런스 문서 (22개)
│       ├── README.md            ← 메인 인덱스
│       ├── common/              ← 개발환경, 프로젝트 구조, 배포
│       ├── 2025/                ← 어셈블리 버전, CHANGELOG
│       ├── 2026/                ← 어셈블리 버전, CHANGELOG
│       ├── autocad-api/         ← Transaction, DB, Entity, Selection, Geometry
│       ├── civil3d-api/         ← Alignment, Profile, Surface, Corridor, Pipe
│       ├── code-samples/basic/  ← Hello Civil3D 예제
│       └── appendix/            ← DXF, 시스템변수, 외부링크
└── (Phase 1~7 폴더 - 추후 생성)
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

## 연관 저장소
- **Csharp-Learning**: C# 학습 프로젝트
  - `csharp-master-plan.md`: Month 5-6에 api-reference 활용 체크리스트

## 파일럿 프로젝트 로드맵
```
Phase 1: 도면 파일 현황 조회
Phase 2: 업무 체크리스트
Phase 3: 토목 구조물 수량 계산기
Phase 4: 구조물 유형별 설계 계산기
Phase 5: 설계 데이터 조회/분석
Phase 6: 수량 산출서 Excel 출력
Phase 7: 암거 자동배치 프로토타입 ⭐
```

## 최근 작업 (2025-03-24)
- API 레퍼런스 문서 체계 구축 완료 (22개 문서)
- appendix 섹션 추가 (DXF, 시스템변수, 외부링크)
- README.md에 .NET 8.0 정보 반영

## 다음 세션 시작 시
```
"CLAUDE.md 읽고 현재 상태 파악해줘"
```

---
*최종 업데이트: 2025-03-24*
