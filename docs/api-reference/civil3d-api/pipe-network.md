# Pipe Network (관망) API

> Civil 3D 관망 객체 접근

## 개요

Pipe Network는 파이프와 구조물(맨홀 등)로 구성된 배관 시스템을 모델링합니다.

## 네임스페이스

```csharp
using Autodesk.Civil.DatabaseServices;
```

## 관망 구조

```
Network (관망)
├── Pipes (파이프)
│   └── Pipe
└── Structures (구조물)
    └── Structure
```

## 관망 조회

```csharp
public void GetAllNetworks()
{
    CivilDocument civilDoc = CivilApplication.ActiveDocument;
    Database db = doc.Database;

    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        ObjectIdCollection networkIds = civilDoc.GetPipeNetworkIds();

        foreach (ObjectId id in networkIds)
        {
            Network network = tr.GetObject(id, OpenMode.ForRead) as Network;

            doc.Editor.WriteMessage($"\n관망: {network.Name}");
            doc.Editor.WriteMessage($"  - 파이프 수: {network.GetPipeIds().Count}");
            doc.Editor.WriteMessage($"  - 구조물 수: {network.GetStructureIds().Count}");
        }

        tr.Commit();
    }
}
```

## 파이프 정보

```csharp
public void GetPipeInfo(Network network, Transaction tr)
{
    ObjectIdCollection pipeIds = network.GetPipeIds();

    foreach (ObjectId pipeId in pipeIds)
    {
        Pipe pipe = tr.GetObject(pipeId, OpenMode.ForRead) as Pipe;

        doc.Editor.WriteMessage($"\n파이프: {pipe.Name}");
        doc.Editor.WriteMessage($"  - 길이: {pipe.Length2D:F2}m");
        doc.Editor.WriteMessage($"  - 직경(내경): {pipe.InnerDiameterOrWidth:F0}mm");
        doc.Editor.WriteMessage($"  - 경사: {pipe.Slope:F4}");
        doc.Editor.WriteMessage($"  - 시작 구조물: {pipe.StartStructureId}");
        doc.Editor.WriteMessage($"  - 종료 구조물: {pipe.EndStructureId}");
    }
}
```

## 구조물 정보

```csharp
public void GetStructureInfo(Network network, Transaction tr)
{
    ObjectIdCollection structureIds = network.GetStructureIds();

    foreach (ObjectId structId in structureIds)
    {
        Structure structure = tr.GetObject(structId, OpenMode.ForRead) as Structure;

        doc.Editor.WriteMessage($"\n구조물: {structure.Name}");
        doc.Editor.WriteMessage($"  - 위치: ({structure.Position.X:F2}, {structure.Position.Y:F2})");
        doc.Editor.WriteMessage($"  - 림(Rim) 표고: {structure.RimElevation:F3}m");
        doc.Editor.WriteMessage($"  - 바닥(Sump) 표고: {structure.SumpElevation:F3}m");
        doc.Editor.WriteMessage($"  - 연결 파이프 수: {structure.GetConnectedPipeIds().Count}");
    }
}
```

## 파이프 속성

| 속성 | 타입 | 설명 |
|------|------|------|
| Name | string | 파이프 이름 |
| Length2D | double | 2D 길이 |
| Length3D | double | 3D 길이 |
| InnerDiameterOrWidth | double | 내경 또는 폭 |
| OuterDiameterOrWidth | double | 외경 또는 폭 |
| Slope | double | 경사 |
| StartPoint | Point3d | 시작점 |
| EndPoint | Point3d | 종점 |
| StartStructureId | ObjectId | 시작 구조물 |
| EndStructureId | ObjectId | 종료 구조물 |

## 구조물 속성

| 속성 | 타입 | 설명 |
|------|------|------|
| Name | string | 구조물 이름 |
| Position | Point3d | 위치 |
| RimElevation | double | 림 표고 |
| SumpElevation | double | 바닥 표고 |
| Rotation | double | 회전 각도 |

## 연결 관계 조회

```csharp
public void TraceNetwork(Structure startStructure, Transaction tr)
{
    Queue<ObjectId> queue = new Queue<ObjectId>();
    HashSet<ObjectId> visited = new HashSet<ObjectId>();

    queue.Enqueue(startStructure.ObjectId);
    visited.Add(startStructure.ObjectId);

    while (queue.Count > 0)
    {
        ObjectId currentId = queue.Dequeue();
        Structure current = tr.GetObject(currentId, OpenMode.ForRead) as Structure;

        doc.Editor.WriteMessage($"\n구조물: {current.Name}");

        foreach (ObjectId pipeId in current.GetConnectedPipeIds())
        {
            Pipe pipe = tr.GetObject(pipeId, OpenMode.ForRead) as Pipe;
            doc.Editor.WriteMessage($"\n  → 파이프: {pipe.Name}");

            // 연결된 다음 구조물
            ObjectId nextStructId = pipe.StartStructureId == currentId
                ? pipe.EndStructureId
                : pipe.StartStructureId;

            if (!visited.Contains(nextStructId) && !nextStructId.IsNull)
            {
                visited.Add(nextStructId);
                queue.Enqueue(nextStructId);
            }
        }
    }
}
```

## 압력관망 (Pressure Network)

```csharp
using Autodesk.Civil.DatabaseServices;
using Autodesk.Civil.PressurePipes;

public void GetPressureNetworks()
{
    CivilDocument civilDoc = CivilApplication.ActiveDocument;

    ObjectIdCollection pressureNetIds = civilDoc.GetPressurePipeNetworkIds();

    // 압력관망 처리...
}
```

## 주의사항

- 파이프와 구조물은 Network를 통해 관리됨
- 연결 관계 수정 시 양쪽 객체 모두 업데이트 필요
- 압력관망은 별도의 API 사용 (AeccPressurePipesMgd.dll)

---

*관련: [Surface API](./surface.md) | [Alignment API](./alignment.md)*
