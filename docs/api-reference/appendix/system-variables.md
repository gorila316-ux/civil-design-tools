# 개발자용 시스템 변수

> AutoCAD/Civil 3D 플러그인 개발 시 알아야 할 주요 시스템 변수

## 보안 관련

### SECURELOAD

플러그인 로드 보안 수준을 제어합니다.

| 값 | 설명 |
|-----|------|
| 0 | 모든 위치에서 실행 파일 로드 (경고 없음) |
| 1 | 신뢰할 수 있는 위치에서만 로드 (기본값) |
| 2 | 신뢰할 수 있는 위치에서만 로드 + 경고 표시 |

```
명령: SECURELOAD
새 값 입력 <1>: 0
```

> ⚠️ **주의**: 개발 중에만 0으로 설정하고, 배포 시에는 1 또는 2 유지

### TRUSTEDPATHS

신뢰할 수 있는 폴더 경로 목록입니다.

```
명령: TRUSTEDPATHS
현재 값: "C:\MyPlugins;C:\AnotherPath"
```

**프로그래밍 방식 설정:**
```csharp
Application.SetSystemVariable("TRUSTEDPATHS",
    @"C:\MyPlugins;C:\AnotherPath");
```

### TRUSTEDDOMAINS

신뢰할 수 있는 URL 도메인 목록입니다.

## 로드/실행 관련

### DEMANDLOAD

ObjectARX 응용프로그램 수요 로드를 제어합니다.

| 값 | 설명 |
|-----|------|
| 0 | 수요 로드 끔 |
| 1 | 사용자 지정 객체 감지 시 로드 |
| 2 | 명령 호출 시 로드 |
| 3 | 둘 다 (기본값) |

### APPAUTOLOAD

시작 시 자동 로드되는 응용프로그램을 제어합니다.

| 비트 | 값 | 설명 |
|------|-----|------|
| 1 | 1 | acad.lsp 로드 |
| 2 | 2 | acaddoc.lsp 로드 |
| 4 | 4 | 도면별 .lsp 로드 |

## 디버깅 관련

### CMDECHO

명령 프롬프트 표시를 제어합니다.

| 값 | 설명 |
|-----|------|
| 0 | 프롬프트 숨김 (스크립트용) |
| 1 | 프롬프트 표시 (기본값) |

### FILEDIA

파일 대화상자 표시를 제어합니다.

| 값 | 설명 |
|-----|------|
| 0 | 대화상자 숨김 (명령행 입력) |
| 1 | 대화상자 표시 (기본값) |

### NOMUTT

메시지 표시를 억제합니다.

| 값 | 설명 |
|-----|------|
| 0 | 메시지 표시 (기본값) |
| 1 | 메시지 억제 |

## 성능 관련

### INDEXCTL

도면 인덱스 생성을 제어합니다.

| 값 | 설명 |
|-----|------|
| 0 | 인덱스 없음 |
| 1 | 도면층 인덱스 생성 |
| 2 | 공간 인덱스 생성 |
| 3 | 둘 다 생성 |

### REGENMODE

자동 재생성을 제어합니다.

| 값 | 설명 |
|-----|------|
| 0 | REGEN 수동 |
| 1 | REGEN 자동 (기본값) |

## 좌표계 관련

### WORLDUCS

현재 UCS가 WCS인지 확인합니다 (읽기 전용).

| 값 | 설명 |
|-----|------|
| 0 | 현재 UCS ≠ WCS |
| 1 | 현재 UCS = WCS |

### UCSFOLLOW

UCS 변경 시 자동 평면 뷰 전환을 제어합니다.

## 단위 관련

### LUNITS

선형 단위 형식입니다.

| 값 | 형식 |
|-----|------|
| 1 | 과학적 |
| 2 | 소수점 |
| 3 | 엔지니어링 |
| 4 | 건축 |
| 5 | 분수 |

### LUPREC

선형 단위 정밀도 (소수점 자릿수)입니다.

### AUNITS

각도 단위 형식입니다.

| 값 | 형식 |
|-----|------|
| 0 | 소수점 도 |
| 1 | 도/분/초 |
| 2 | 그레이드 |
| 3 | 라디안 |
| 4 | 측량 단위 |

## .NET에서 시스템 변수 사용

### 값 읽기

```csharp
// 정수 값
int secureload = Convert.ToInt32(
    Application.GetSystemVariable("SECURELOAD"));

// 문자열 값
string paths = Application.GetSystemVariable("TRUSTEDPATHS") as string;

// Point3d 값
Point3d lastPoint = (Point3d)Application.GetSystemVariable("LASTPOINT");
```

### 값 설정

```csharp
// 정수 값
Application.SetSystemVariable("SECURELOAD", 0);

// 문자열 값
Application.SetSystemVariable("TRUSTEDPATHS", @"C:\MyPath");

// OSMODE (객체 스냅)
Application.SetSystemVariable("OSMODE", 0);  // 끄기
Application.SetSystemVariable("OSMODE", 39); // 끝점+중간점+중심점
```

### 임시 변경 패턴

```csharp
public void ExecuteWithoutPrompts()
{
    // 현재 값 저장
    object oldCmdEcho = Application.GetSystemVariable("CMDECHO");
    object oldFileDia = Application.GetSystemVariable("FILEDIA");

    try
    {
        // 임시 변경
        Application.SetSystemVariable("CMDECHO", 0);
        Application.SetSystemVariable("FILEDIA", 0);

        // 작업 수행
        doc.SendStringToExecute("_ZOOM _E ", true, false, false);
    }
    finally
    {
        // 원래 값 복원
        Application.SetSystemVariable("CMDECHO", oldCmdEcho);
        Application.SetSystemVariable("FILEDIA", oldFileDia);
    }
}
```

## 객체 스냅 (OSMODE) 비트값

| 비트 | 값 | 스냅 모드 |
|------|-----|----------|
| 1 | 1 | 끝점 (ENDpoint) |
| 2 | 2 | 중간점 (MIDpoint) |
| 4 | 4 | 중심 (CENter) |
| 8 | 8 | 노드 (NODe) |
| 16 | 16 | 사분점 (QUAdrant) |
| 32 | 32 | 교차점 (INTersection) |
| 64 | 64 | 삽입점 (INSertion) |
| 128 | 128 | 수직 (PERpendicular) |
| 256 | 256 | 접선 (TANgent) |
| 512 | 512 | 근접점 (NEArest) |
| 1024 | 1024 | 외관 교차 |
| 2048 | 2048 | 확장 (EXTension) |
| 4096 | 4096 | 평행 (PARallel) |

**예: 끝점 + 중간점 + 중심 = 1 + 2 + 4 = 7**

## 참조 링크

- [SECURELOAD (Autodesk Help)](https://help.autodesk.com/cloudhelp/2017/ENU/AutoCAD-Core/files/GUID-541566C6-2738-49DD-87C3-C1490E924A02.htm)
- [System Variables Reference](https://help.autodesk.com/view/ACD/2025/ENU/?guid=GUID-E0F71A30-29D7-4B77-A489-3E1C16E5E8EB)
- [AutoCAD SECURELOAD (Autodesk Blog)](https://blog.autodesk.io/all-you-need-to-know-about-autocad-secureload-au/)

---

*관련: [Development Setup](../common/development-setup.md) | [Plugin Bundle](../common/plugin-bundle.md)*
