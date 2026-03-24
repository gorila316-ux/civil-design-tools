# DXF 그룹 코드 참조

> AutoCAD DXF 파일 형식의 그룹 코드 빠른 참조

## 개요

DXF(Drawing Exchange Format)는 AutoCAD 도면 데이터의 텍스트 기반 표현입니다. 각 데이터 항목은 **그룹 코드**와 **값**의 쌍으로 구성됩니다.

## 공통 그룹 코드

### 엔티티 공통 (0-99)

| 코드 | 설명 | 예시 |
|------|------|------|
| 0 | 엔티티 타입 | "LINE", "CIRCLE", "LWPOLYLINE" |
| 5 | 핸들 (고유 ID) | "1A3" |
| 6 | 선종류 이름 | "CONTINUOUS", "DASHED" |
| 7 | 문자 스타일 이름 | "Standard" |
| 8 | 도면층 이름 | "0", "MyLayer" |
| 10 | 기본 X 좌표 | 100.0 |
| 20 | 기본 Y 좌표 | 50.0 |
| 30 | 기본 Z 좌표 | 0.0 |
| 11-18 | 추가 X 좌표 | - |
| 21-28 | 추가 Y 좌표 | - |
| 31-38 | 추가 Z 좌표 | - |
| 39 | 두께 (Thickness) | 0.0 |
| 40-48 | 실수 값 (반지름, 축척 등) | - |
| 50-58 | 각도 (도 단위) | - |
| 62 | 색상 번호 | 1 (빨강), 256 (ByLayer) |
| 67 | 모형/도면 공간 | 0=모형, 1=도면 |
| 70-78 | 정수 값 (플래그 등) | - |

### 확장 데이터 (1000-1071)

| 코드 | 설명 |
|------|------|
| 1000 | 문자열 (최대 255자) |
| 1001 | 등록된 응용프로그램 이름 |
| 1010, 1020, 1030 | 3D 점 |
| 1040 | 실수 값 |
| 1070 | 16비트 정수 |
| 1071 | 32비트 정수 |

## 주요 엔티티 그룹 코드

### LINE (선)

```
0       <- 엔티티 타입
LINE
8       <- 도면층
MyLayer
10      <- 시작점 X
0.0
20      <- 시작점 Y
0.0
30      <- 시작점 Z
0.0
11      <- 끝점 X
100.0
21      <- 끝점 Y
50.0
31      <- 끝점 Z
0.0
```

| 코드 | 필수 | 설명 |
|------|------|------|
| 10, 20, 30 | O | 시작점 (X, Y, Z) |
| 11, 21, 31 | O | 끝점 (X, Y, Z) |
| 39 | X | 두께 |

### CIRCLE (원)

| 코드 | 필수 | 설명 |
|------|------|------|
| 10, 20, 30 | O | 중심점 (X, Y, Z) |
| 40 | O | 반지름 |
| 39 | X | 두께 |
| 210, 220, 230 | X | 돌출 방향 |

### ARC (호)

| 코드 | 필수 | 설명 |
|------|------|------|
| 10, 20, 30 | O | 중심점 (X, Y, Z) |
| 40 | O | 반지름 |
| 50 | O | 시작 각도 (도) |
| 51 | O | 끝 각도 (도) |

### LWPOLYLINE (경량 폴리선)

| 코드 | 필수 | 설명 |
|------|------|------|
| 90 | O | 정점 수 |
| 70 | X | 폴리선 플래그 (1=닫힘) |
| 43 | X | 일정 폭 |
| 10, 20 | O | 정점 좌표 (반복) |
| 42 | X | Bulge (곡률) |
| 40 | X | 시작 폭 |
| 41 | X | 끝 폭 |

**폴리선 플래그 (코드 70) 비트값:**

| 비트 | 값 | 설명 |
|------|-----|------|
| 1 | 1 | 닫힌 폴리선 |
| 128 | 128 | Plinegen |

### TEXT (문자)

| 코드 | 필수 | 설명 |
|------|------|------|
| 10, 20, 30 | O | 삽입점 |
| 40 | O | 문자 높이 |
| 1 | O | 문자열 내용 |
| 50 | X | 회전 각도 (도) |
| 41 | X | 상대 X 축척 |
| 51 | X | 기울기 각도 |
| 7 | X | 문자 스타일 이름 |
| 71 | X | 문자 생성 플래그 |
| 72 | X | 수평 정렬 |
| 73 | X | 수직 정렬 |
| 11, 21, 31 | X | 정렬점 (72, 73 != 0일 때) |

### INSERT (블록 참조)

| 코드 | 필수 | 설명 |
|------|------|------|
| 2 | O | 블록 이름 |
| 10, 20, 30 | O | 삽입점 |
| 41 | X | X 축척 (기본 1.0) |
| 42 | X | Y 축척 (기본 1.0) |
| 43 | X | Z 축척 (기본 1.0) |
| 50 | X | 회전 각도 (도) |
| 66 | X | 속성 따름 플래그 |

### HATCH (해치)

| 코드 | 필수 | 설명 |
|------|------|------|
| 2 | O | 패턴 이름 |
| 70 | O | 솔리드 채움 플래그 (1=솔리드) |
| 71 | O | 연관 플래그 |
| 91 | O | 경계 경로 수 |
| 75 | X | 해치 스타일 |
| 76 | X | 패턴 타입 |

## .NET에서 DXF 코드 사용

### SelectionFilter 예제

```csharp
// 빨간색 선만 선택
TypedValue[] filter = new TypedValue[]
{
    new TypedValue((int)DxfCode.Start, "LINE"),      // 코드 0
    new TypedValue((int)DxfCode.Color, 1)            // 코드 62 = 빨강
};
SelectionFilter sf = new SelectionFilter(filter);
```

### DxfCode 열거형 (주요 값)

```csharp
DxfCode.Start = 0           // 엔티티 타입
DxfCode.Text = 1            // 문자열
DxfCode.XRefPath = 1        // Xref 경로
DxfCode.LayerName = 8       // 도면층
DxfCode.LinetypeName = 6    // 선종류
DxfCode.Color = 62          // 색상
DxfCode.Operator = -4       // 필터 연산자 ("<AND", "AND>", 등)
```

## 색상 코드 참조

| 코드 | 색상 |
|------|------|
| 0 | ByBlock |
| 1 | 빨강 |
| 2 | 노랑 |
| 3 | 초록 |
| 4 | 청록 |
| 5 | 파랑 |
| 6 | 자홍 |
| 7 | 흰색/검정 |
| 256 | ByLayer |

## 참조 링크

- [AutoCAD 2025 DXF Reference](https://help.autodesk.com/cloudhelp/2025/ENU/AutoCAD-DXF/files/GUID-A35B8C2A-1885-4A8E-8533-E61D8A423D62.htm)
- [DXF Group Codes (AfraLISP)](https://www.afralisp.net/reference/dxf-group-codes.php)
- [AutoCAD DXF Archive (Autodesk)](https://damassets.autodesk.net/content/dam/autodesk/www/developer-network/platform-technologies/autocad-dxf-archive/)

---

*관련: [Entity Manipulation](../autocad-api/entity-manipulation.md) | [Selection](../autocad-api/selection.md)*
