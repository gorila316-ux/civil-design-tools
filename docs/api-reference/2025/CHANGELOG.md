# Civil 3D 2025 API 변경사항

> 2024 대비 2025 버전의 API 변경 내역

## 개요

| 항목 | 값 |
|------|-----|
| 출시일 | 2024년 3월 |
| ObjectARX 버전 | 2025 |
| AeccDbMgd 버전 | 14.0.0.0 |
| **.NET 플랫폼** | **.NET 8.0** (주요 변경) |

## 주요 변경: .NET 8.0 전환

### Breaking Change

Civil 3D 2025부터 **.NET Framework 4.8 → .NET 8.0**으로 전환되었습니다.

```
기존 (.NET Framework 4.8)     →    신규 (.NET 8.0)
- AutoCAD 2024 이하               - AutoCAD 2025 이상
- .NET Framework 전용 API 사용 가능  - 일부 API 제거/변경
```

### 마이그레이션 필수 작업

1. **프로젝트 대상 프레임워크 변경**
   ```xml
   <!-- 기존 -->
   <TargetFramework>net48</TargetFramework>

   <!-- 신규 -->
   <TargetFramework>net8.0-windows</TargetFramework>
   ```

2. **제거된 API 대체**
   - `Marshal.GetActiveObject()` → 수동 COM 구현 필요
   - `AppDomain` 관련 API 제한

3. **재컴파일 필수**
   - .NET Framework 4.8용 DLL은 2025에서 로드 불가
   - 모든 플러그인 재빌드 필요

## Civil 3D 신규 기능 API

### Corridor (코리더) 개선

| 기능 | 설명 |
|------|------|
| 다중 Baseline 선택 | 코리더 생성 시 여러 선형/종단 동시 선택 가능 |
| Feature Line 필터링 | Name, Style, Layer, Site로 필터링 |
| 코드셋 스타일 | 생성 시점에 코드셋 스타일 적용 가능 |
| Transition Set 복사 | 같은 코리더 또는 다른 도면으로 복사/붙여넣기 |

**성능 최적화:**
- 코리더가 out-of-date일 때 불필요한 Surface/Section 재계산 방지
- Feature Line 재생성 효율 개선

### Surface (지형) 개선

| 기능 | 설명 |
|------|------|
| 불필요한 재생성 방지 | 이름/설명 변경 시 지형 재생성 안 함 |
| Level of Detail API | 지형 LOD 기능 API 지원 |

### Alignment (선형) 수정

| 수정 | 설명 |
|------|------|
| StationOffsetAcceptOutOfRange() | StackOverflowException 발생 버그 수정 |

## Deprecated (지원 중단 예정)

| API | 상태 | 대체 방법 |
|-----|------|-----------|
| .NET Framework 전용 API | 제거됨 | .NET 8 호환 API 사용 |
| Marshal.GetActiveObject | 제거됨 | 수동 COM 구현 |

## 호환성 참고

### 하위 호환성

```
❌ 2025 플러그인 → 2024 이하에서 실행 불가
❌ 2024 플러그인 → 2025에서 실행 불가 (재컴파일 필요)
```

### 권장 개발 환경

| 도구 | 버전 |
|------|------|
| Visual Studio | 2022 (17.8 이상) |
| .NET SDK | 8.0 |
| C# | 12.0 |

## 참조 링크

- [.NET Migration Guide](https://help.autodesk.com/view/OARX/2025/ENU/?guid=OARX-ManagedRefGuide-_NET_Migration_Guide)
- [ObjectARX 2025 Documentation](https://help.autodesk.com/view/OARX/2025/ENU/)
- [Migrating to .NET 8 (CAD Guardian)](https://www.cadguardian.com/blog/migrating-your-autocad-net-add-ins-from-2023-to-2025-(-net-8))
- [Civil 3D 2025 What's New (AUGI)](https://www.augi.com/articles/detail/autodesk-autocad-civil3d-2025-whats-new-in-civil-3d-2025)

---

*최종 업데이트: 2026-03-24*
