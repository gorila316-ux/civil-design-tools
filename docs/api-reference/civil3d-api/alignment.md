# Alignment (선형) API

> Civil 3D 선형 객체 접근 및 조작

## 개요

선형(Alignment)은 Civil 3D의 핵심 객체로, 도로, 철도 등의 중심선을 정의합니다.

## 네임스페이스

```csharp
using Autodesk.Civil.DatabaseServices;
using Autodesk.Civil.ApplicationServices;
```

## 클래스 계층

```
Entity (AutoCAD)
└── Autodesk.Civil.DatabaseServices.Entity
    └── Alignment
```

## 선형 가져오기

### 모든 선형 조회

```csharp
public void GetAllAlignments()
{
    Document doc = Application.DocumentManager.MdiActiveDocument;
    Database db = doc.Database;
    CivilDocument civilDoc = CivilApplication.ActiveDocument;

    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        // 선형 컬렉션 가져오기
        ObjectIdCollection alignmentIds = civilDoc.GetAlignmentIds();

        foreach (ObjectId alignId in alignmentIds)
        {
            Alignment alignment = tr.GetObject(alignId, OpenMode.ForRead) as Alignment;

            if (alignment != null)
            {
                doc.Editor.WriteMessage($"\n선형: {alignment.Name}");
                doc.Editor.WriteMessage($"  - 길이: {alignment.Length:F2}m");
                doc.Editor.WriteMessage($"  - 시작 스테이션: {alignment.StartingStation:F2}");
                doc.Editor.WriteMessage($"  - 종료 스테이션: {alignment.EndingStation:F2}");
            }
        }

        tr.Commit();
    }
}
```

### 이름으로 선형 찾기

```csharp
public Alignment GetAlignmentByName(CivilDocument civilDoc, Transaction tr, string name)
{
    ObjectIdCollection alignmentIds = civilDoc.GetAlignmentIds();

    foreach (ObjectId id in alignmentIds)
    {
        Alignment alignment = tr.GetObject(id, OpenMode.ForRead) as Alignment;
        if (alignment?.Name == name)
            return alignment;
    }

    return null;
}
```

## 선형 속성

| 속성 | 타입 | 설명 |
|------|------|------|
| Name | string | 선형 이름 |
| Length | double | 전체 길이 |
| StartingStation | double | 시작 스테이션 |
| EndingStation | double | 종료 스테이션 |
| ReferencePointStation | double | 기준점 스테이션 |
| ReferencePoint | Point2d | 기준점 좌표 |

## 스테이션/오프셋 작업

### 좌표 → 스테이션/오프셋

```csharp
public void GetStationOffset(Alignment alignment, Point3d point)
{
    double station = 0;
    double offset = 0;

    alignment.StationOffset(point.X, point.Y, ref station, ref offset);

    doc.Editor.WriteMessage($"\n스테이션: {station:F2}, 오프셋: {offset:F2}");
}
```

### 스테이션/오프셋 → 좌표

```csharp
public Point2d GetPointAtStationOffset(Alignment alignment, double station, double offset)
{
    double x = 0, y = 0;

    alignment.PointLocation(station, offset, ref x, ref y);

    return new Point2d(x, y);
}
```

## 선형 엔티티 (곡선 요소)

```csharp
public void AnalyzeAlignmentEntities(Alignment alignment)
{
    AlignmentEntityCollection entities = alignment.Entities;

    foreach (AlignmentEntity entity in entities)
    {
        switch (entity.EntityType)
        {
            case AlignmentEntityType.Line:
                AlignmentLine line = entity as AlignmentLine;
                doc.Editor.WriteMessage($"\n직선: 길이={line.Length:F2}");
                break;

            case AlignmentEntityType.Arc:
                AlignmentArc arc = entity as AlignmentArc;
                doc.Editor.WriteMessage($"\n원곡선: R={arc.Radius:F2}, L={arc.Length:F2}");
                break;

            case AlignmentEntityType.Spiral:
                AlignmentSpiral spiral = entity as AlignmentSpiral;
                doc.Editor.WriteMessage($"\n완화곡선: L={spiral.Length:F2}");
                break;
        }
    }
}
```

## 종단 (Profile) 연결

```csharp
public void GetProfiles(Alignment alignment, Transaction tr)
{
    ObjectIdCollection profileIds = alignment.GetProfileIds();

    foreach (ObjectId profileId in profileIds)
    {
        Profile profile = tr.GetObject(profileId, OpenMode.ForRead) as Profile;
        doc.Editor.WriteMessage($"\n종단: {profile.Name}");
    }
}
```

## 주의사항

- 선형 수정 시 반드시 `OpenMode.ForWrite`로 열기
- 트랜잭션 내에서 작업 필수
- 대용량 선형 처리 시 성능 고려

---

*관련: [Profile API](./profile.md) | [Corridor API](./corridor.md)*
