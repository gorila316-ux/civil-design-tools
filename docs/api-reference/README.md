# Civil 3D API Reference

> Civil 3D .NET API 개발을 위한 레퍼런스 문서

## 지원 버전

| 버전 | 상태 | ObjectARX SDK | .NET Framework |
|------|------|---------------|----------------|
| Civil 3D 2025 | ✅ 지원 | 2025 | .NET 4.8 |
| Civil 3D 2026 | ✅ 지원 | 2026 | .NET 4.8 |

## 문서 구조

```
api-reference/
├── common/                 # 버전 공통 문서
│   ├── development-setup.md
│   ├── project-structure.md
│   └── plugin-bundle.md
│
├── 2025/                   # Civil 3D 2025 전용
│   ├── CHANGELOG.md
│   ├── assembly-versions.md
│   └── breaking-changes.md
│
├── 2026/                   # Civil 3D 2026 전용
│   ├── CHANGELOG.md
│   ├── assembly-versions.md
│   └── new-features.md
│
├── autocad-api/            # AutoCAD 기반 API
│   ├── database-access.md
│   ├── entity-manipulation.md
│   └── transaction.md
│
├── civil3d-api/            # Civil 3D 객체 API
│   ├── alignment.md
│   ├── profile.md
│   ├── surface.md
│   ├── corridor.md
│   └── pipe-network.md
│
└── code-samples/           # 코드 예제
    ├── basic/
    └── version-specific/
```

## 빠른 시작

1. [개발환경 설정](./common/development-setup.md) - Visual Studio + Civil 3D 설정
2. [프로젝트 구조](./common/project-structure.md) - 솔루션 구성 방법
3. [기본 예제](./code-samples/basic/) - Hello World 수준 예제

## 버전 선택 가이드

### 신규 프로젝트
- **2026 권장**: 최신 API, 장기 지원

### 기존 프로젝트 유지보수
- 기존 버전 유지 또는 점진적 마이그레이션 권장
- [Breaking Changes](./2026/breaking-changes.md) 확인 필수

## 외부 참조 링크

### 공식 문서
- [ObjectARX 2025 온라인 문서](https://help.autodesk.com/view/OARX/2025/KOR/)
- [ObjectARX 2026 온라인 문서](https://help.autodesk.com/view/OARX/2026/KOR/)
- [Autodesk Developer Network](https://www.autodesk.com/developer-network)

### 커뮤니티
- [Autodesk Forums](https://forums.autodesk.com)
- [Through the Interface (Kean Walmsley)](https://through-the-interface.typepad.com/)
- [ADN DevBlog](https://adndevblog.typepad.com/autocad/)

---

*문서 생성일: 2026-03-24*
