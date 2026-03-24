# Civil 3D 2026 API 변경사항

> 2025 대비 2026 버전의 API 변경 내역

## 개요

| 항목 | 값 |
|------|-----|
| 출시일 | 2025년 3월 |
| ObjectARX 버전 | 2026 |
| AeccDbMgd 버전 | 15.0.0.0 |
| .NET 플랫폼 | .NET 8.0 (2025와 동일) |

## 2025 대비 변경사항

### 어셈블리 버전 변경

| 어셈블리 | 2025 | 2026 |
|----------|------|------|
| acdbmgd.dll | 25.0.0.0 | 26.0.0.0 |
| acmgd.dll | 25.0.0.0 | 26.0.0.0 |
| accoremgd.dll | 25.0.0.0 | 26.0.0.0 |
| AeccDbMgd.dll | 14.0.0.0 | 15.0.0.0 |
| AecBaseMgd.dll | 25.0.0.0 | 26.0.0.0 |

### 플랫폼 요구사항

2026은 2025와 동일하게 .NET 8.0을 사용합니다.

```xml
<TargetFramework>net8.0-windows</TargetFramework>
```

## Breaking Changes

### 1. 어셈블리 재컴파일 필요

2025용으로 빌드된 플러그인은 2026에서 로드되지 않습니다.
- 참조 DLL을 2026 버전으로 교체
- 재컴파일 필수

### 2. COM Interop 주의사항 (2025부터 적용)

.NET 8에서 제거된 API들은 2026에서도 동일하게 적용됩니다:

```csharp
// ❌ 사용 불가 (.NET 8에서 제거)
object app = Marshal.GetActiveObject("AutoCAD.Application");

// ✅ 대안: ROT(Running Object Table) 직접 접근
// 또는 Process 기반 접근
```

## 신규 기능

### Civil 3D 2026 신기능

*공식 릴리즈 노트 확인 후 업데이트 예정*

| 기능 | API 지원 | 설명 |
|------|----------|------|
| TBD | TBD | TBD |

## 마이그레이션 가이드

### 2025 → 2026 마이그레이션

1. **참조 업데이트**
   ```
   C:\Program Files\Autodesk\AutoCAD 2025\
   → C:\Program Files\Autodesk\AutoCAD 2026\
   ```

2. **프로젝트 파일 수정** (필요시)
   ```xml
   <ItemGroup>
     <Reference Include="AeccDbMgd">
       <HintPath>C:\Program Files\Autodesk\AutoCAD 2026\C3D\AeccDbMgd.dll</HintPath>
     </Reference>
   </ItemGroup>
   ```

3. **재컴파일 및 테스트**
   - Release 모드로 빌드
   - 주요 기능 테스트

### 호환성 체크리스트

- [ ] 모든 Autodesk DLL 참조를 2026 버전으로 변경
- [ ] .NET 8.0 대상 확인
- [ ] x64 플랫폼 대상 확인
- [ ] Deprecated API 사용 여부 확인
- [ ] COM Interop 코드 검토
- [ ] 테스트 환경에서 NETLOAD 테스트
- [ ] 번들 배포 테스트

## 멀티 버전 지원

### 조건부 컴파일

```csharp
#if CIVIL3D2026
    // 2026 전용 코드
#elif CIVIL3D2025
    // 2025 전용 코드
#endif
```

### 프로젝트 구성

```xml
<PropertyGroup Condition="'$(Configuration)'=='Release2025'">
    <DefineConstants>CIVIL3D2025</DefineConstants>
</PropertyGroup>
<PropertyGroup Condition="'$(Configuration)'=='Release2026'">
    <DefineConstants>CIVIL3D2026</DefineConstants>
</PropertyGroup>
```

## 참조 링크

- [ObjectARX 2026 Documentation](https://help.autodesk.com/view/OARX/2026/ENU/)
- [.NET Migration Guide](https://help.autodesk.com/view/OARX/2026/ENU/?guid=OARX-ManagedRefGuide-_NET_Migration_Guide)
- [About Managed .NET Compatibility](https://help.autodesk.com/view/OARX/2026/ENU/?guid=GUID-A6C680F2-DE2E-418A-A182-E4884073338A)

---

*최종 업데이트: 2026-03-24*
