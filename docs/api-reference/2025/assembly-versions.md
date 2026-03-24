# Civil 3D 2025 어셈블리 버전

> Civil 3D 2025 개발 시 참조해야 할 어셈블리 정보

## AutoCAD 기본 어셈블리

| 어셈블리 | 버전 | 설명 |
|----------|------|------|
| accoremgd.dll | 25.0.0.0 | AutoCAD Core Managed |
| acdbmgd.dll | 25.0.0.0 | AutoCAD Database Managed |
| acmgd.dll | 25.0.0.0 | AutoCAD Managed |
| AcWindows.dll | 25.0.0.0 | AutoCAD Windows UI |

**경로**: `C:\Program Files\Autodesk\AutoCAD 2025\`

## Civil 3D 어셈블리

| 어셈블리 | 버전 | 설명 |
|----------|------|------|
| AeccDbMgd.dll | 14.0.0.0 | Civil 3D Database |
| AeccPressurePipesMgd.dll | 14.0.0.0 | 압력관망 |
| AecBaseMgd.dll | 25.0.0.0 | AEC 기본 |

**경로**: `C:\Program Files\Autodesk\AutoCAD 2025\C3D\`

## 프로젝트 참조 설정

### .csproj 예제

```xml
<ItemGroup>
    <!-- AutoCAD -->
    <Reference Include="accoremgd">
        <HintPath>C:\Program Files\Autodesk\AutoCAD 2025\accoremgd.dll</HintPath>
        <Private>False</Private>
        <SpecificVersion>False</SpecificVersion>
    </Reference>
    <Reference Include="acdbmgd">
        <HintPath>C:\Program Files\Autodesk\AutoCAD 2025\acdbmgd.dll</HintPath>
        <Private>False</Private>
    </Reference>
    <Reference Include="acmgd">
        <HintPath>C:\Program Files\Autodesk\AutoCAD 2025\acmgd.dll</HintPath>
        <Private>False</Private>
    </Reference>

    <!-- Civil 3D -->
    <Reference Include="AeccDbMgd">
        <HintPath>C:\Program Files\Autodesk\AutoCAD 2025\C3D\AeccDbMgd.dll</HintPath>
        <Private>False</Private>
    </Reference>
    <Reference Include="AecBaseMgd">
        <HintPath>C:\Program Files\Autodesk\AutoCAD 2025\C3D\AecBaseMgd.dll</HintPath>
        <Private>False</Private>
    </Reference>
</ItemGroup>
```

## ObjectARX SDK

- **다운로드**: [Autodesk Developer Network](https://www.autodesk.com/developer-network/platform-technologies/autocad/objectarx)
- **온라인 문서**: [ObjectARX 2025 Documentation](https://help.autodesk.com/view/OARX/2025/KOR/)

## 주요 네임스페이스

```csharp
// AutoCAD
Autodesk.AutoCAD.Runtime
Autodesk.AutoCAD.ApplicationServices
Autodesk.AutoCAD.DatabaseServices
Autodesk.AutoCAD.EditorInput
Autodesk.AutoCAD.Geometry

// Civil 3D
Autodesk.Civil.ApplicationServices
Autodesk.Civil.DatabaseServices
Autodesk.Civil.DatabaseServices.Styles
Autodesk.Civil.DataShortcuts
Autodesk.Civil.Settings
```

---

*참조: [2026 어셈블리 버전](../2026/assembly-versions.md)*
