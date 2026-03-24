# Corridor (코리더) API

> Civil 3D 코리더 객체 접근

## 개요

코리더는 선형, 종단, 어셈블리를 결합하여 3D 도로/철도 모델을 생성합니다.

## 네임스페이스

```csharp
using Autodesk.Civil.DatabaseServices;
```

## 코리더 구조

```
Corridor
├── Baselines (기준선)
│   └── Baseline
│       ├── BaselineRegions (구간)
│       │   └── BaselineRegion
│       │       └── AppliedAssembly (적용된 어셈블리)
│       └── FeatureLines (특성선)
└── CorridorSurfaces (코리더 지형)
```

## 코리더 조회

```csharp
public void GetAllCorridors()
{
    CivilDocument civilDoc = CivilApplication.ActiveDocument;
    Database db = doc.Database;

    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        ObjectIdCollection corridorIds = civilDoc.GetCorridorIds();

        foreach (ObjectId id in corridorIds)
        {
            Corridor corridor = tr.GetObject(id, OpenMode.ForRead) as Corridor;

            doc.Editor.WriteMessage($"\n코리더: {corridor.Name}");
            doc.Editor.WriteMessage($"  - Baseline 수: {corridor.Baselines.Count}");
        }

        tr.Commit();
    }
}
```

## Baseline 정보

```csharp
public void GetBaselineInfo(Corridor corridor)
{
    foreach (Baseline baseline in corridor.Baselines)
    {
        doc.Editor.WriteMessage($"\nBaseline: {baseline.Name}");
        doc.Editor.WriteMessage($"  - 선형: {baseline.AlignmentId}");
        doc.Editor.WriteMessage($"  - 종단: {baseline.ProfileId}");
        doc.Editor.WriteMessage($"  - Region 수: {baseline.BaselineRegions.Count}");

        foreach (BaselineRegion region in baseline.BaselineRegions)
        {
            doc.Editor.WriteMessage($"\n    Region: {region.Name}");
            doc.Editor.WriteMessage($"      시작: STA {region.StartStation:F2}");
            doc.Editor.WriteMessage($"      종료: STA {region.EndStation:F2}");
            doc.Editor.WriteMessage($"      어셈블리: {region.AssemblyId}");
        }
    }
}
```

## 특성선 (Feature Line) 접근

```csharp
public void GetFeatureLines(Corridor corridor)
{
    foreach (Baseline baseline in corridor.Baselines)
    {
        FeatureLineCollection featureLines = baseline.MainBaselineFeatureLines.FeatureLineCollectionMap;

        foreach (FeatureLineCollection flColl in featureLines)
        {
            doc.Editor.WriteMessage($"\n특성선 그룹: {flColl.FeatureLineCodeInfo.CodeName}");

            foreach (CorridorFeatureLine fl in flColl)
            {
                doc.Editor.WriteMessage($"\n  - 특성선: {fl.CodeName}");
            }
        }
    }
}
```

## 코리더 지형

```csharp
public void GetCorridorSurfaces(Corridor corridor)
{
    foreach (CorridorSurface corrSurf in corridor.CorridorSurfaces)
    {
        doc.Editor.WriteMessage($"\n코리더 지형: {corrSurf.Name}");
        doc.Editor.WriteMessage($"  - Surface ID: {corrSurf.SurfaceId}");
    }
}
```

## 코리더 재생성

```csharp
public void RebuildCorridor(Corridor corridor)
{
    // corridor는 OpenMode.ForWrite로 열어야 함
    corridor.Rebuild();
}
```

## 횡단 정보 추출

```csharp
public void GetCrossSectionData(Corridor corridor, double station)
{
    foreach (Baseline baseline in corridor.Baselines)
    {
        foreach (BaselineRegion region in baseline.BaselineRegions)
        {
            if (station >= region.StartStation && station <= region.EndStation)
            {
                // 해당 스테이션의 적용된 어셈블리
                AppliedAssembly appliedAsm = region.AppliedAssemblies
                    .FirstOrDefault(a => Math.Abs(a.AppliedAtStation - station) < 0.01);

                if (appliedAsm != null)
                {
                    foreach (AppliedSubassembly subAsm in appliedAsm.GetAppliedSubassemblies())
                    {
                        doc.Editor.WriteMessage($"\n  서브어셈블리: {subAsm.SubassemblyId}");

                        foreach (CalculatedPoint point in subAsm.CalculatedPoints)
                        {
                            doc.Editor.WriteMessage($"\n    Point: {point.StationOffsetElevationToBaseline}");
                        }
                    }
                }
            }
        }
    }
}
```

## 주의사항

- 코리더는 대용량 객체로 처리 시간 고려 필요
- `Rebuild()` 호출은 시간이 오래 걸릴 수 있음
- 코리더 수정 후 관련 지형 업데이트 필요

---

*관련: [Alignment API](./alignment.md) | [Profile API](./profile.md)*
