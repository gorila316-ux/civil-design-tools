# Civil 3D 2026 어셈블리 버전

> Civil 3D 2026 개발 시 참조해야 할 어셈블리 정보

## AutoCAD 기본 어셈블리

| 어셈블리 | 버전 | 설명 |
|----------|------|------|
| accoremgd.dll | 26.0.0.0 | AutoCAD Core Managed |
| acdbmgd.dll | 26.0.0.0 | AutoCAD Database Managed |
| acmgd.dll | 26.0.0.0 | AutoCAD Managed |
| AcWindows.dll | 26.0.0.0 | AutoCAD Windows UI |

**경로**: `C:\Program Files\Autodesk\AutoCAD 2026\`

## Civil 3D 어셈블리

| 어셈블리 | 버전 | 설명 |
|----------|------|------|
| AeccDbMgd.dll | 15.0.0.0 | Civil 3D Database |
| AeccPressurePipesMgd.dll | 15.0.0.0 | 압력관망 |
| AecBaseMgd.dll | 26.0.0.0 | AEC 기본 |

**경로**: `C:\Program Files\Autodesk\AutoCAD 2026\C3D\`

## 2025 대비 변경사항

| 어셈블리 | 2025 버전 | 2026 버전 | 변경 |
|----------|-----------|-----------|------|
| acdbmgd.dll | 25.0.0.0 | 26.0.0.0 | Major +1 |
| AeccDbMgd.dll | 14.0.0.0 | 15.0.0.0 | Major +1 |

## 프로젝트 참조 설정

### .csproj 예제

```xml
<ItemGroup>
    <!-- AutoCAD -->
    <Reference Include="accoremgd">
        <HintPath>C:\Program Files\Autodesk\AutoCAD 2026\accoremgd.dll</HintPath>
        <Private>False</Private>
        <SpecificVersion>False</SpecificVersion>
    </Reference>
    <Reference Include="acdbmgd">
        <HintPath>C:\Program Files\Autodesk\AutoCAD 2026\acdbmgd.dll</HintPath>
        <Private>False</Private>
    </Reference>
    <Reference Include="acmgd">
        <HintPath>C:\Program Files\Autodesk\AutoCAD 2026\acmgd.dll</HintPath>
        <Private>False</Private>
    </Reference>

    <!-- Civil 3D -->
    <Reference Include="AeccDbMgd">
        <HintPath>C:\Program Files\Autodesk\AutoCAD 2026\C3D\AeccDbMgd.dll</HintPath>
        <Private>False</Private>
    </Reference>
    <Reference Include="AecBaseMgd">
        <HintPath>C:\Program Files\Autodesk\AutoCAD 2026\C3D\AecBaseMgd.dll</HintPath>
        <Private>False</Private>
    </Reference>
</ItemGroup>
```

## 멀티 버전 프로젝트 구성

### 조건부 참조

```xml
<PropertyGroup>
    <Civil3DVersion Condition="'$(Civil3DVersion)'==''">2026</Civil3DVersion>
</PropertyGroup>

<ItemGroup Condition="'$(Civil3DVersion)'=='2025'">
    <Reference Include="AeccDbMgd">
        <HintPath>C:\Program Files\Autodesk\AutoCAD 2025\C3D\AeccDbMgd.dll</HintPath>
    </Reference>
</ItemGroup>

<ItemGroup Condition="'$(Civil3DVersion)'=='2026'">
    <Reference Include="AeccDbMgd">
        <HintPath>C:\Program Files\Autodesk\AutoCAD 2026\C3D\AeccDbMgd.dll</HintPath>
    </Reference>
</ItemGroup>
```

## ObjectARX SDK

- **다운로드**: [Autodesk Developer Network](https://www.autodesk.com/developer-network/platform-technologies/autocad/objectarx)
- **온라인 문서**: [ObjectARX 2026 Documentation](https://help.autodesk.com/view/OARX/2026/KOR/)

---

*참조: [2025 어셈블리 버전](../2025/assembly-versions.md)*
