# 플러그인 번들 배포

> AutoCAD/Civil 3D 플러그인 패키징 및 배포 방법

## 번들 구조

```
MyPlugin.bundle/
├── PackageContents.xml          # 필수: 번들 메타데이터
└── Contents/
    ├── MyPlugin.dll             # 메인 어셈블리
    ├── MyPlugin.UI.dll          # UI 어셈블리 (있는 경우)
    └── Resources/               # 리소스 (아이콘 등)
```

## PackageContents.xml 예제

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationPackage
    SchemaVersion="1.0"
    Name="MyPlugin"
    Description="Civil 3D 플러그인 설명"
    Author="작성자"
    Version="1.0.0"
    ProductCode="{GUID-HERE}">

    <CompanyDetails
        Name="회사명"
        Email="email@example.com"/>

    <RuntimeRequirements
        SupportPath="./Contents"
        OS="Win64"
        Platform="AutoCAD|Civil3D"
        SeriesMin="R24.0"
        SeriesMax="R25.0"/>

    <Components>
        <!-- Civil 3D 2025 -->
        <ComponentEntry
            AppName="MyPlugin_2025"
            ModuleName="./Contents/MyPlugin.dll"
            AppDescription="MyPlugin for Civil 3D 2025"
            LoadOnAutoCADStartup="True"
            LoadOnCommandInvocation="False"
            AppType=".Net">
            <Commands>
                <Command Local="MYCOMMAND" Global="MYCOMMAND"/>
            </Commands>
        </ComponentEntry>

        <!-- Civil 3D 2026 (버전별 분리 시) -->
        <ComponentEntry
            AppName="MyPlugin_2026"
            Version="26.0"
            ModuleName="./Contents/2026/MyPlugin.dll"
            AppDescription="MyPlugin for Civil 3D 2026"
            LoadOnAutoCADStartup="True"
            AppType=".Net">
        </ComponentEntry>
    </Components>

</ApplicationPackage>
```

## 설치 경로

### 사용자별 설치 (권장)
```
%APPDATA%\Autodesk\ApplicationPlugins\MyPlugin.bundle\
```

### 시스템 전체 설치
```
C:\ProgramData\Autodesk\ApplicationPlugins\MyPlugin.bundle\
```

## 멀티 버전 지원

### 방법 1: 단일 DLL (권장)

동일한 DLL이 여러 버전에서 작동하도록 설계:

```xml
<RuntimeRequirements
    SeriesMin="R24.0"   <!-- 2025 -->
    SeriesMax="R26.0"/> <!-- 2026 -->
```

### 방법 2: 버전별 DLL

```
Contents/
├── 2025/
│   └── MyPlugin.dll
└── 2026/
    └── MyPlugin.dll
```

```xml
<ComponentEntry Version="25.0" ModuleName="./Contents/2025/MyPlugin.dll" .../>
<ComponentEntry Version="26.0" ModuleName="./Contents/2026/MyPlugin.dll" .../>
```

## 배포 체크리스트

- [ ] PackageContents.xml 유효성 검증
- [ ] 모든 의존성 DLL 포함 (Autodesk DLL 제외)
- [ ] x64 플랫폼 빌드 확인
- [ ] .NET Framework 4.8 대상 확인
- [ ] Release 모드 빌드
- [ ] 테스트 환경에서 NETLOAD 테스트
- [ ] 번들 폴더에 복사 후 자동 로드 테스트

## 디버깅 팁

### 로드 실패 시 확인사항

1. Civil 3D 명령행에서 `NETLOAD` 수동 실행
2. Windows 이벤트 뷰어에서 .NET 오류 확인
3. PackageContents.xml 문법 오류 확인

### 로그 활성화

```
SECURELOAD = 0  (개발 중에만, 보안 주의)
```

---

*관련: [개발환경 설정](./development-setup.md)*
