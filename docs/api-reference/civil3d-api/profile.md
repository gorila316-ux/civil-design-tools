# Profile (종단) API

> Civil 3D 종단 객체 접근 및 조작

## 개요

Profile(종단)은 선형을 따라 수직 방향의 설계를 정의합니다. 현황 종단과 설계 종단으로 구분됩니다.

## 네임스페이스

```csharp
using Autodesk.Civil.DatabaseServices;
```

## Profile 유형

| 유형 | 설명 |
|------|------|
| EG (Existing Ground) | 현황 종단 - 지형에서 추출 |
| FG (Finished Grade) | 설계 종단 - 직접 설계 |
| Superimposed | 중첩 종단 |

## 종단 조회

### 선형에서 종단 가져오기

```csharp
public void GetProfilesFromAlignment(Alignment alignment, Transaction tr)
{
    ObjectIdCollection profileIds = alignment.GetProfileIds();

    foreach (ObjectId profileId in profileIds)
    {
        Profile profile = tr.GetObject(profileId, OpenMode.ForRead) as Profile;

        doc.Editor.WriteMessage($"\n종단: {profile.Name}");
        doc.Editor.WriteMessage($"  - 유형: {profile.ProfileType}");
        doc.Editor.WriteMessage($"  - 시작 스테이션: {profile.StartingStation:F2}");
        doc.Editor.WriteMessage($"  - 종료 스테이션: {profile.EndingStation:F2}");
    }
}
```

## 종단 표고 조회

### 특정 스테이션의 표고

```csharp
public double GetElevationAtStation(Profile profile, double station)
{
    try
    {
        return profile.ElevationAt(station);
    }
    catch (Exception)
    {
        // 스테이션이 종단 범위를 벗어난 경우
        return double.NaN;
    }
}
```

### 구간별 표고 조회

```csharp
public void GetElevationsAtIntervals(Profile profile, double interval)
{
    double startSta = profile.StartingStation;
    double endSta = profile.EndingStation;

    for (double sta = startSta; sta <= endSta; sta += interval)
    {
        double elev = profile.ElevationAt(sta);
        doc.Editor.WriteMessage($"\nSTA {sta:F2} → EL {elev:F3}");
    }
}
```

## 종단 속성

| 속성 | 타입 | 설명 |
|------|------|------|
| Name | string | 종단 이름 |
| ProfileType | ProfileType | 종단 유형 |
| StartingStation | double | 시작 스테이션 |
| EndingStation | double | 종료 스테이션 |
| MinElevation | double | 최저 표고 |
| MaxElevation | double | 최고 표고 |

## 종단 엔티티 (설계 요소)

```csharp
public void AnalyzeProfileEntities(Profile profile)
{
    ProfileEntityCollection entities = profile.Entities;

    foreach (ProfileEntity entity in entities)
    {
        switch (entity.EntityType)
        {
            case ProfileEntityType.Tangent:
                ProfileTangent tangent = entity as ProfileTangent;
                doc.Editor.WriteMessage($"\n직선: 경사={tangent.Grade:F2}%");
                break;

            case ProfileEntityType.Circular:
                ProfileCircular circular = entity as ProfileCircular;
                doc.Editor.WriteMessage($"\n원곡선: R={circular.Radius:F2}");
                break;

            case ProfileEntityType.ParabolicSymmetric:
                ProfileParabolicSymmetric parabola = entity as ProfileParabolicSymmetric;
                doc.Editor.WriteMessage($"\n포물선: L={parabola.Length:F2}");
                break;
        }
    }
}
```

## PVI (종단 변곡점) 작업

```csharp
public void GetPVIData(Profile profile)
{
    ProfilePVICollection pvis = profile.PVIs;

    doc.Editor.WriteMessage($"\nPVI 개수: {pvis.Count}");

    foreach (ProfilePVI pvi in pvis)
    {
        doc.Editor.WriteMessage($"\n  STA: {pvi.Station:F2}, EL: {pvi.Elevation:F3}");
    }
}
```

## 현황 종단 생성 (지형에서)

```csharp
public ObjectId CreateEGProfile(
    ObjectId alignmentId,
    ObjectId surfaceId,
    ObjectId styleId,
    ObjectId labelSetId)
{
    return Profile.CreateFromSurface(
        "현황종단",
        alignmentId,
        surfaceId,
        styleId,
        labelSetId
    );
}
```

## 설계 종단 생성

```csharp
public ObjectId CreateFGProfile(
    ObjectId alignmentId,
    ObjectId styleId,
    ObjectId labelSetId)
{
    return Profile.CreateByLayout(
        "설계종단",
        alignmentId,
        styleId,
        labelSetId
    );
}
```

## 주의사항

- 종단은 항상 선형에 종속됨
- `ElevationAt()` 메서드는 범위 외 스테이션에서 예외 발생
- 설계 종단 수정 시 해당 코리더 재생성 필요

---

*관련: [Alignment API](./alignment.md) | [Corridor API](./corridor.md)*
