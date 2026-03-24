# Surface (지형) API

> Civil 3D 지형 객체 접근 및 조작

## 개요

Surface는 Civil 3D의 지형 데이터를 표현하며, TIN Surface와 Grid Surface 등 여러 유형이 있습니다.

## 네임스페이스

```csharp
using Autodesk.Civil.DatabaseServices;
using Autodesk.Civil.DatabaseServices.Styles;
```

## Surface 유형

| 유형 | 클래스 | 설명 |
|------|--------|------|
| TIN Surface | TinSurface | 삼각망 기반 지형 |
| Grid Surface | GridSurface | 격자 기반 지형 |
| Volume Surface | TinVolumeSurface | 토량 계산용 |

## 지형 조회

### 모든 지형 가져오기

```csharp
public void GetAllSurfaces()
{
    Document doc = Application.DocumentManager.MdiActiveDocument;
    Database db = doc.Database;
    CivilDocument civilDoc = CivilApplication.ActiveDocument;

    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        ObjectIdCollection surfaceIds = civilDoc.GetSurfaceIds();

        foreach (ObjectId surfId in surfaceIds)
        {
            Surface surface = tr.GetObject(surfId, OpenMode.ForRead) as Surface;

            if (surface != null)
            {
                doc.Editor.WriteMessage($"\n지형: {surface.Name}");

                if (surface is TinSurface tinSurf)
                {
                    doc.Editor.WriteMessage($"  - 유형: TIN");
                    doc.Editor.WriteMessage($"  - 점 개수: {tinSurf.Vertices.Count}");
                    doc.Editor.WriteMessage($"  - 삼각형 개수: {tinSurf.Triangles.Count}");
                }
            }
        }

        tr.Commit();
    }
}
```

## TinSurface 주요 속성

| 속성 | 타입 | 설명 |
|------|------|------|
| Name | string | 지형 이름 |
| Vertices | TinSurfaceVertexCollection | 정점 컬렉션 |
| Triangles | TinSurfaceTriangleCollection | 삼각형 컬렉션 |
| MinimumElevation | double | 최저 표고 |
| MaximumElevation | double | 최고 표고 |

## 표고 조회

### 특정 좌표의 표고

```csharp
public double? GetElevationAt(TinSurface surface, double x, double y)
{
    try
    {
        double elevation = surface.FindElevationAtXY(x, y);
        return elevation;
    }
    catch (Exception)
    {
        // 지형 범위 밖인 경우
        return null;
    }
}
```

### 다수 포인트 표고 조회

```csharp
public void GetElevationsAtPoints(TinSurface surface, List<Point2d> points)
{
    foreach (Point2d pt in points)
    {
        try
        {
            double elev = surface.FindElevationAtXY(pt.X, pt.Y);
            doc.Editor.WriteMessage($"\n({pt.X:F2}, {pt.Y:F2}) → 표고: {elev:F3}");
        }
        catch
        {
            doc.Editor.WriteMessage($"\n({pt.X:F2}, {pt.Y:F2}) → 범위 외");
        }
    }
}
```

## 지형 통계

```csharp
public void GetSurfaceStatistics(TinSurface surface)
{
    GeneralSurfaceProperties props = surface.GetGeneralProperties();

    doc.Editor.WriteMessage($"\n=== {surface.Name} 통계 ===");
    doc.Editor.WriteMessage($"\n최소 표고: {surface.MinimumElevation:F3}");
    doc.Editor.WriteMessage($"\n최대 표고: {surface.MaximumElevation:F3}");
    doc.Editor.WriteMessage($"\n2D 면적: {props.Area2d:F2} m²");
    doc.Editor.WriteMessage($"\n3D 면적: {props.Area3d:F2} m²");
}
```

## 지형 편집

### 점 추가

```csharp
public void AddPointToSurface(TinSurface surface, Point3d point)
{
    // surface는 OpenMode.ForWrite로 열어야 함
    surface.AddVertex(point);
}
```

### 경계선 추가

```csharp
public void AddBoundary(TinSurface surface, ObjectId polylineId)
{
    surface.BoundariesDefinition.AddBoundaries(
        new ObjectIdCollection { polylineId },
        1.0,  // MidOrdinateDistance
        SurfaceBoundaryType.Outer,
        true  // UseNonDestructiveBreakline
    );
}
```

## Volume Surface (토량 계산)

```csharp
public void CreateVolumeSurface(CivilDocument civilDoc,
    ObjectId baseSurfaceId, ObjectId comparisonSurfaceId)
{
    // 토량 지형 생성
    ObjectId volSurfId = TinVolumeSurface.Create(
        civilDoc,
        "토량 지형",
        baseSurfaceId,
        comparisonSurfaceId
    );

    // 토량 조회
    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        TinVolumeSurface volSurf = tr.GetObject(volSurfId, OpenMode.ForRead) as TinVolumeSurface;

        doc.Editor.WriteMessage($"\n절토량: {volSurf.GetVolumeProperties().UnadjustedCutVolume:F2} m³");
        doc.Editor.WriteMessage($"\n성토량: {volSurf.GetVolumeProperties().UnadjustedFillVolume:F2} m³");
        doc.Editor.WriteMessage($"\n순토량: {volSurf.GetVolumeProperties().UnadjustedNetVolume:F2} m³");

        tr.Commit();
    }
}
```

## 주의사항

- 대용량 지형 처리 시 메모리 관리 필요
- `FindElevationAtXY`는 지형 범위 외부에서 예외 발생
- 지형 수정 후 `Rebuild()` 호출 권장

---

*관련: [Alignment API](./alignment.md) | [Profile API](./profile.md)*
