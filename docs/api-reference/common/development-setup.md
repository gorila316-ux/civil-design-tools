# 개발환경 설정

> Civil 3D .NET 플러그인 개발을 위한 환경 구성

## 필수 소프트웨어

| 소프트웨어 | 버전 | 비고 |
|-----------|------|------|
| Visual Studio | 2022 이상 | Community 가능 |
| .NET Framework | 4.8 | Civil 3D 요구사항 |
| Civil 3D | 2025 또는 2026 | 개발/테스트용 |

## Visual Studio 설정

### 1. 워크로드 설치

Visual Studio Installer에서 다음 워크로드 선택:
- **.NET 데스크톱 개발**
- **C++를 사용한 데스크톱 개발** (ObjectARX 네이티브 디버깅 시)

### 2. 프로젝트 생성

```
새 프로젝트 > 클래스 라이브러리(.NET Framework) > .NET Framework 4.8
```

### 3. 참조 추가

Civil 3D 설치 경로에서 다음 DLL 참조:

```
C:\Program Files\Autodesk\AutoCAD 2025\
├── accoremgd.dll        # AutoCAD Core
├── acdbmgd.dll          # AutoCAD Database
├── acmgd.dll            # AutoCAD Managed
└── ...

C:\Program Files\Autodesk\AutoCAD 2025\C3D\
├── AeccDbMgd.dll        # Civil 3D Database
├── AeccPressurePipesMgd.dll
└── ...
```

### 4. 참조 속성 설정

모든 Autodesk DLL 참조에 대해:
- **로컬 복사**: `False`
- **특정 버전**: `False`

## 디버깅 설정

### 프로젝트 속성 > 디버그

```
시작 작업: 외부 프로그램 시작
프로그램: C:\Program Files\Autodesk\AutoCAD 2025\acad.exe
```

### 디버그 시작 인수 (선택)

```
/nologo /b "startup.scr"
```

## 빌드 후 이벤트 (선택)

자동으로 플러그인 폴더에 복사:

```batch
xcopy "$(TargetPath)" "C:\Users\%USERNAME%\AppData\Roaming\Autodesk\ApplicationPlugins\MyPlugin.bundle\Contents\" /Y
```

## 문제 해결

### DLL 버전 충돌

```xml
<!-- app.config에 바인딩 리디렉션 추가 -->
<configuration>
  <runtime>
    <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
      <dependentAssembly>
        <assemblyIdentity name="AcDbMgd" publicKeyToken="null" culture="neutral"/>
        <bindingRedirect oldVersion="0.0.0.0-25.0.0.0" newVersion="25.0.0.0"/>
      </dependentAssembly>
    </assemblyBinding>
  </runtime>
</configuration>
```

### Civil 3D가 DLL을 로드하지 않음

1. `NETLOAD` 명령으로 수동 로드 테스트
2. 대상 프레임워크 확인 (.NET 4.8)
3. 플랫폼 대상: `x64`

---

*다음: [프로젝트 구조](./project-structure.md)*
